# Populate

MongoDB在> = 3.2版本中具有类似连接的$ lookup聚合操作符。 Mongoose有一个更强大的替代方法叫做populate（），它可以让你引用其他集合中的文档。

Population 是自动将文档中的指定路径替换为其他集合中的文档的过程。我们可以填充单个文档，多个文档，普通对象，多个普通对象或从查询返回的所有对象。我们来看一些例子。

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var personSchema = Schema({
  _id: Schema.Types.ObjectId,
  name: String,
  age: Number,
  stories: [{ type: Schema.Types.ObjectId, ref: 'Story' }]
});

var storySchema = Schema({
  author: { type: Schema.Types.ObjectId, ref: 'Person' },
  title: String,
  fans: [{ type: Schema.Types.ObjectId, ref: 'Person' }]
});

var Story = mongoose.model('Story', storySchema);
var Person = mongoose.model('Person', personSchema);
```

到目前为止，我们创建了两个模型。我们的Person模型的 stories 字段设置为一个ObjectIds数组。 ref选项告诉Mongoose在 population 中使用哪个模型，在我们的案例中是Story模型。我们在这里存储的所有_id必须是来自Story模型的document _ids。

注意：ObjectId， Number ，String和Buffer可用作 ref。但是，除非您是高级用户，并且有充分理由这么做，否则应该使用ObjectId。

## Saving refs

将 ref 保存到其他文档的工作方式与您通常保存属性的方式相同，只需指定_id值即可：

```js
var author = new Person({
  _id: new mongoose.Types.ObjectId(),
  name: 'Ian Fleming',
  age: 50
});

author.save(function (err) {
  if (err) return handleError(err);

  var story1 = new Story({
    title: 'Casino Royale',
    author: author._id    // assign the _id from the person
  });

  story1.save(function (err) {
    if (err) return handleError(err);
    // thats it!
  });
});
```

## Population

到目前为止，我们没有做太多的改变。我们只创建了一个 Person和一个Story。现在让我们看看使用 query builder 填充 story 的 author ：

```js
Story.
  findOne({ title: 'Casino Royale' }).
  populate('author').
  exec(function (err, story) {
    if (err) return handleError(err);
    console.log('The author is %s', story.author.name);
    // prints "The author is Ian Fleming"
  });
```

填充的路径不再设置为它们的原始 _id，它们的值由之前单独执行查询从数据库返回的mongoose文档替换。

refs 数组 的工作方式相同。只需在查询中调用 populate 方法，并将返回一组文档替换原始文档_id。

## 设置填充字段

在Mongoose> = 4.0中，您也可以手动填充字段。

```js
Story.findOne({ title: 'Casino Royale' }, function(error, story) {
  if (error) {
    return handleError(error);
  }
  story.author = author;
  console.log(story.author.name); // prints "Ian Fleming"
});
```

## 字段选择

如果我们只想为填充文档返回一些特定字段，该怎么办？这可以通过将通常的 字段名称语法作为第二个参数传递给填充方法来完成：

```js
Story.
  findOne({ title: /casino royale/i }).
  populate('author', 'name'). // only return the Persons name
  exec(function (err, story) {
    if (err) return handleError(err);

    console.log('The author is %s', story.author.name);
    // prints "The author is Ian Fleming"

    console.log('The authors age is %s', story.author.age);
    // prints "The authors age is null'
  });
```

## 填充多个路径

如果我们想要同时填充多条路径会怎么样？

```js
Story.
  find(...).
  populate('fans').
  populate('author').
  exec();

```

如果您populate()使用相同的路径多次呼叫，则只有最后一个会生效。
```js
// The 2nd `populate()` call below overwrites the first because they
// both populate 'fans'.
Story.
  find().
  populate({ path: 'fans', select: 'name' }).
  populate({ path: 'fans', select: 'email' });
// The above is equivalent to:
Story.find().populate({ path: 'fans', select: 'email' });
```

## 查询条件和其他选项

如果我们想根据他们的年龄填充我们的粉丝阵列，选择他们的名字，最多只能返回5个粉丝？

```js
Story.
  find(...).
  populate({
    path: 'fans',
    match: { age: { $gte: 21 }},
    // Explicitly exclude `_id`, see http://bit.ly/2aEfTdB
    select: 'name -_id',
    options: { limit: 5 }
  }).
  exec();
```

## 指向儿童

但是，如果我们使用该author对象，我们可能会发现我们无法获得这些故事的列表。这是因为没有任何story东西被“推”到author.stories。

这里有两个观点。首先，你可能想author知道哪些故事是他们的。通常情况下，您的模式应该通过在“多边”中有一个父指针来解决一对多关系。但是，如果您有充足的理由需要一组子指针，则可以push()按如下所示将文档编入数组中。

author.stories.push(story1);
author.save(callback);

这使我们能够执行find和populate组合：

```js
Person.
  findOne({ name: 'Ian Fleming' }).
  populate('stories'). // only works if we pushed refs to children
  exec(function (err, person) {
    if (err) return handleError(err);
    console.log(person);
  });
```

有争议的是，我们确实需要两组指针，因为它们可能不同步。相反，我们可以直接跳过find()我们感兴趣的故事。

```js
Story.
  find({ author: author._id }).
  exec(function (err, stories) {
    if (err) return handleError(err);
    console.log('The stories are an array: ', stories);
  });
```

除非指定了精益选项，否则从查询群体返回的文档将 变为功能齐全且功能强大的文档 。不要将它们与子文档混淆。调用它的remove方法时要小心，因为您将从数据库中删除它，而不仅仅是数组。removesave

## 填充现有文档

如果我们有一个现有的猫鼬文档并且想要填充它的一些路径，mongoose> = 3.6支持 文档＃populate（）方法。

## 填充多个现有文档

如果我们有一个或多个猫鼬文档甚至普通对象（类似mapReduce output_），我们可以使用mongoose > = 3.6中 提供的Model.populate（）方法来填充它们。这是什么document#populate() ，并query#populate()用它来填充文件。

## 填充多个级别

假设你有一个跟踪用户朋友的用户模式。

var userSchema = new Schema({
  name: String,
  friends: [{ type: ObjectId, ref: 'User' }]
});
填充让你得到一个用户朋友的列表，但是如果你还想要一个用户朋友的朋友呢？指定populate告诉猫鼬填充friends所有用户朋友的数组的选项：
```js
User.
  findOne({ name: 'Val' }).
  populate({
    path: 'friends',
    // Get friends of friends - populate the 'friends' array for every friend
    populate: { path: 'friends' }
  });
```

## 填充数据库

假设您有一个表示事件的模式和一个表示对话的模式。每个事件都有相应的对话线程。
```js
var eventSchema = new Schema({
  name: String,
  // The id of the corresponding conversation
  // Notice there's no ref here!
  conversation: ObjectId
});
var conversationSchema = new Schema({
  numMessages: Number
});
```

另外，假设事件和对话存储在不同的MongoDB实例中。
```js
var db1 = mongoose.createConnection('localhost:27000/db1');
var db2 = mongoose.createConnection('localhost:27001/db2');

var Event = db1.model('Event', eventSchema);
var Conversation = db2.model('Conversation', conversationSchema);
```

在这种情况下，你会不会能够populate()正常进行。该conversation字段将始终为空，因为populate()不知道要使用哪个模型。但是， 您可以明确指定模型。
```js
Event.
  find().
  populate({ path: 'conversation', model: Conversation }).
  exec(function(error, docs) { /* ... */ });

```

这被称为“跨数据库填充”，因为它使您能够跨MongoDB数据库甚至跨MongoDB实例填充。

## 动态参考

Mongoose也可以同时从多个集合中填充。假设您有一个具有“连接”阵列的用户架构 - 用户可以连接到其他用户或组织。

```js
var userSchema = new Schema({
  name: String,
  connections: [{
    kind: String,
    item: { type: ObjectId, refPath: 'connections.kind' }
  }]
});

var organizationSchema = new Schema({ name: String, kind: String });

var User = mongoose.model('User', userSchema);
var Organization = mongoose.model('Organization', organizationSchema);
```

上述refPath属性意味着猫鼬会查看 connections.kind路径以确定使用哪种模型populate()。换句话说，该refPath属性使您可以使ref 属性动态。

```js
// Say we have one organization:
// `{ _id: ObjectId('000000000000000000000001'), name: "Guns N' Roses", kind: 'Band' }`
// And two users:
// {
//   _id: ObjectId('000000000000000000000002')
//   name: 'Axl Rose',
//   connections: [
//     { kind: 'User', item: ObjectId('000000000000000000000003') },
//     { kind: 'Organization', item: ObjectId('000000000000000000000001') }
//   ]
// },
// {
//   _id: ObjectId('000000000000000000000003')
//   name: 'Slash',
//   connections: []
// }
User.
  findOne({ name: 'Axl Rose' }).
  populate('connections.item').
  exec(function(error, doc) {
    // doc.connections[0].item is a User doc
    // doc.connections[1].item is an Organization doc
  });
```

## 填充虚拟

4.5.0新增功能

到目前为止，您只基于该_id字段进行填充。但是，这有时不是正确的选择。特别是，无边界增长的数组是MongoDB的反模式。使用猫鼬虚拟，你可以定义更复杂的文件之间的关系。

```js
var PersonSchema = new Schema({
  name: String,
  band: String
});

var BandSchema = new Schema({
  name: String
});
BandSchema.virtual('members', {
  ref: 'Person', // The model to use
  localField: 'name', // Find people where `localField`
  foreignField: 'band', // is equal to `foreignField`
  // If `justOne` is true, 'members' will be a single doc as opposed to
  // an array. `justOne` is false by default.
  justOne: false
});

var Person = mongoose.model('Person', PersonSchema);
var Band = mongoose.model('Band', BandSchema);

/**
 * Suppose you have 2 bands: "Guns N' Roses" and "Motley Crue"
 * And 4 people: "Axl Rose" and "Slash" with "Guns N' Roses", and
 * "Vince Neil" and "Nikki Sixx" with "Motley Crue"
 */
Band.find({}).populate('members').exec(function(error, bands) {
  /* `bands.members` is now an array of instances of `Person` */
});
```

请记住，虚拟输出不包括在内toJSON()

默认。如果您想在使用依赖JSON.stringify()的res.json()函数（如Express 函数）时显示填充虚拟，请virtuals: true在架构选项上设置toJSON选项。

```js
// Set `virtuals: true` so `res.json()` works
var BandSchema = new Schema({
  name: String
}, { toJSON: { virtuals: true } });
```

如果您使用填充投影，请确保foreignField包含在投影中。

```js
Band.
  find({}).
  populate({ path: 'members', select: 'name' }).
  exec(function(error, bands) {
    // Won't work, foreign field `band` is not selected in the projection
  });

Band.
  find({}).
  populate({ path: 'members', select: 'name band' }).
  exec(function(error, bands) {
    // Works, foreign field `band` is selected
  });
```

## 填充在中间件中


您可以填入前或后的钩子。如果您想始终填充某个字段，请查看mongoose-autopopulate插件。

```js
// Always attach `populate()` to `find()` calls
MySchema.pre('find', function() {
  this.populate('user');
});
// Always `populate()` after `find()` calls. Useful if you want to selectively populate
// based on the docs found.
MySchema.post('find', async function(docs) {
  for (let doc of docs) {
    if (doc.isPublic) {
      await doc.populate('user').execPopulate();
    }
  }
});
// `populate()` after saving. Useful for sending populated data back to the client in an
// update API endpoint
MySchema.post('save', function(doc, next) {
  doc.populate('user').execPopulate(function() {
    next();
  });
});
```
