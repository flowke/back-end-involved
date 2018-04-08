# Schemas

## 定义一个 schema

Mongoose中的所有内容都以Schema开头。每个 schema 映射一个MongoDB集合, 并定义该集合中文档的字段。

```js
var mongoose = require('mongoose');
  var Schema = mongoose.Schema;

  var blogSchema = new Schema({
    title:  String,
    author: String,
    body:   String,
    comments: [{ body: String, date: Date }],
    date: { type: Date, default: Date.now },
    hidden: Boolean,
    meta: {
      votes: Number,
      favs:  Number
    }
  });
```

如果想稍后再添加 key, 使用 [ Schema#add](http://mongoosejs.com/docs/api.html#schema_Schema-add) 方法.

定义每一个属性都对应一直数据类型。

合法的类型有:

- String
- Number
- Date
- Buffer
- Boolean
- Mixed
- ObjectId
- Array

查看更多 [Schema类型](http://mongoosejs.com/docs/schematypes.html).

Schemas 不仅用来定义数据结构, 还会定义: 实例方法, 静态 Model 方法, compound indexes, 和 生命周期函数(中间件).

## 创建一个 model

要使用我们定义的 schema, 就需要把它转换成一个 [Model](http://mongoosejs.com/docs/models.html).

使用如下方法:
```js
var Blog = mongoose.model('Blog', blogSchema);
// ready to go!
```

## 实例方法

`Models` 的一个实例就是一个 [documents(文档)](http://mongoosejs.com/docs/documents.html). documents 有很多 [内置实例方法](http://mongoosejs.com/docs/api.html#document-js).

我们也可以定义我们自己的自定义文档实例方法:


```js
// define a schema
var animalSchema = new Schema({ name: String, type: String });

// assign a function to the "methods" object of our animalSchema
animalSchema.methods.findSimilarTypes = function(cb) {
  return this.model('Animal').find({ type: this.type }, cb);
};
```

- 覆盖默认mongoose文档方法可能会导致不可预知的结果。查看 [this](http://mongoosejs.com/docs/api.html#schema_Schema.reserved) 更多细节。
- 不要使用箭头函数. this 指向会有问题.

## 静态方法.

添加静态方法到 `model` 也很容易:

```js
// assign a function to the "statics" object of our animalSchema
animalSchema.statics.findByName = function(name, cb) {
  return this.find({ name: new RegExp(name, 'i') }, cb);
};

var Animal = mongoose.model('Animal', animalSchema);
Animal.findByName('fido', function(err, animals) {
  console.log(animals);
  });
```

## 查询辅助函数

您还可以添加查询辅助函数，这些函数就像实例方法，但是用于 mongoose 查询。查询帮助器方法可让您扩展mongoose的[链式查询构建器API](http://mongoosejs.com/docs/queries.html)。

## 索引

MongoDB支持 [二级索引](http://docs.mongodb.org/manual/indexes/)

## ... 待更新
