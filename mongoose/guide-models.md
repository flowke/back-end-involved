# Models

Model 是根据我们的 schema 定义编译的构造函数。这些 Model 的实例代表可以从我们的数据库中保存和检索的文档。从数据库中创建和检索所有文档都由这些模型处理。

## 编译你的第一个模型

```js
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```

第一个参数是您的 model 所用集合的 '单数' 名称。**mongoose 自动寻找您的模型名称的复数版本**

对于上述示例，模型Tank对应用于数据库中的 tanks 集合。

` .model（）`函数生成 `schema` 的副本。

在调用`.model（）`之前，确保schema已经添加了你想要的所有东西。

## 构建 document

 Document 是 model 的一个实例. 创建并保持到数据库都很容易:

 ```js
 var Tank = mongoose.model('Tank', yourSchema);

 var small = new Tank({ size: 'small' });
 small.save(function (err) {
   if (err) return handleError(err);
   // saved!
 })

 // or

 Tank.create({ size: 'small' }, function (err, small) {
   if (err) return handleError(err);
   // saved!
 })
 ```

 请注意，在您的模型使用的连接打开之前，不会创建/移除 Tanks 。每个模型都有一个关联的连接。当你使用mongoose.model（）时，你的模型将使用默认的mongoose连接:

 ```js
mongoose.connect('localhost', 'gettingstarted');
 ```

如果您创建自定义连接，请改用该连接的 `model()` 函数:

```js
var connection = mongoose.createConnection('mongodb://localhost:27017/test');
var Tank = connection.model('Tank', yourSchema);
```

## 查询

使用Mongoose查找文档很容易，它支持MongoDB的丰富查询语法。可以使用每个模型的find，findById，findOne或静态方法来检索文档。

```js
Tank.find({ size: 'small' }).where('createdDate').gt(oneYearAgo).exec(callback);
```

有关如何使用[Query API](http://mongoosejs.com/docs/api.html#query-js)的更多详细信息，请参阅[查询](http://mongoosejs.com/docs/queries.html)一章。

## 删除

model 有一个静态删除方法可用于删除所有符合匹配条件的 document。

```js
Tank.remove({ size: 'large' }, function (err) {
  if (err) return handleError(err);
  // removed!
});
```

## 更新

每个 `model` 都有自己的 `update` 方法，用于修改数据库中的 document 而不将它们返回给应用程序。有关更多详细信息，请参阅[API文档](http://mongoosejs.com/docs/api.html#model_Model.update)。

> 如果您想更新数据库中的单个文档并将其返回给您的应用程序，请改用 [findOneAndUpdate](http://mongoosejs.com/docs/api.html#model_Model.findOneAndUpdate)
