# 查询

文档可以通过 model 的几种静态 helper 方法来进行查询。

任何只要指定了查询条件的 model 方法, 都可以通过两种方式调用:

1. 通过回调函数
    - 被传递时，操作将立即执行，并将结果传递给回调函数
    - 没有传递，返回一个 `Query` 实例.
2. Query 有一个 `.then()` 方法, 可以当 promise 用.

使用回调函数执行查询时，可以将查询指定为JSON文档。 JSON文档的语法与MongoDB shell相同。

```js
var Person = mongoose.model('Person', yourSchema);

// find each person with a last name matching 'Ghost', selecting the `name` and `occupation` fields
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
  // Prints "Space Ghost is a talk show host".
  console.log('%s %s is a %s.', person.name.first, person.name.last,
    person.occupation);
});
```

在这里，我们看到查询立即执行，结果传递给我们的回调。 Mongoose中的所有回调都使用模式：`callback(error, result)`。如果执行查询时发生错误，错误参数将包含错误文档，并且结果将为 null。如果查询成功，则error参数将为 null，并且结果将填充查询结果。

在回调被传递给Mongoose中的查询的任何地方，回调都遵循模式 :`callback(error, results)`。什么结果取决于操作：对于findOne（），它是一个可能为空的单个文档，find（）文档列表，count（）文档数量，update（）受影响的文档数量等。API文档模型提供了有关传递给回调函数的更多细节。

现在我们来看看没有回调传递时会发生什么：

```js
// find each person with a last name matching 'Ghost'
var query = Person.findOne({ 'name.last': 'Ghost' });

// selecting the `name` and `occupation` fields
query.select('name occupation');

// execute the query at a later time
query.exec(function (err, person) {
  if (err) return handleError(err);
  // Prints "Space Ghost is a talk show host."
  console.log('%s %s is a %s.', person.name.first, person.name.last,
    person.occupation);
});
```

在上面的代码中，查询变量的类型是Query。Query使您能够使用链式语法构建查询，而不是指定JSON对象。以下两个例子是等价的。

```js
// With a JSON doc
Person.
  find({
    occupation: /host/,
    'name.last': 'Ghost',
    age: { $gt: 17, $lt: 66 },
    likes: { $in: ['vaporizing', 'talking'] }
  }).
  limit(10).
  sort({ occupation: -1 }).
  select({ name: 1, occupation: 1 }).
  exec(callback);

// Using query builder
Person.
  find({ occupation: /host/ }).
  where('name.last').equals('Ghost').
  where('age').gt(17).lt(66).
  where('likes').in(['vaporizing', 'talking']).
  limit(10).
  sort('-occupation').
  select('name occupation').
  exec(callback);
```
查询帮助函数的完整列表可以在 [API文档](http://mongoosejs.com/docs/api.html#query-js)中找到。

## 引用其他 documents
MongoDB中没有 joins，但有时我们仍然希望引用其他集合中的文档。这是 [population](http://mongoosejs.com/docs/populate.html) 进入的地方。在这里阅读更多关于如何在其他查询结果中包含来自其他集合的文档。

## Streaming

您可以从MongoDB流式传输查询结果。您需要调用`Query＃cursor（）`函数来返回QueryCursor的一个实例。

```js
var cursor = Person.find({ occupation: /host/ }).cursor();
cursor.on('data', function(doc) {
  // Called once for every document
});
cursor.on('close', function() {
  // Called when done
});
```
