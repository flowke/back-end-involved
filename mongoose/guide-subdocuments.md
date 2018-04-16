# Sub Docs

子文档是嵌入在其他文档中的文档。在Mongoose中，这意味着您可以在其他 schema 中嵌套schema。 Mongoose有两种截然不同的子文档概念：子文档数组和单个嵌套子文档。

```js
var childSchema = new Schema({ name: 'string' });

var parentSchema = new Schema({
  // Array of subdocuments
  children: [childSchema],
  // Single nested subdocuments. Caveat: single nested subdocs only work
  // in mongoose >= 4.2.0
  child: childSchema
});

```
子文档与普通文档类似。嵌套的 schema 可以使用中间件，自定义验证逻辑，virtuals, 以及任何其他顶级schema可以使用的功能。主要区别在于子文档不会单独保存，只要保存其顶级父文档即可保存。

```js
var Parent = mongoose.model('Parent', parentSchema);
var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
parent.children[0].name = 'Matthew';

// `parent.children[0].save()` is a no-op, it triggers middleware but
// does **not** actually save the subdocument. You need to save the parent
// doc.
parent.save(callback);
```

子文档像顶级文档一样有 `save` 和 validate 中间件。在父文档上调用save（）会为其所有子文档触发save（）中间件 和 validate（）中间件。

```js
childSchema.pre('save', function (next) {
  if ('invalid' == this.name) {
    return next(new Error('#sadpanda'));
  }
  next();
});

var parent = new Parent({ children: [{ name: 'invalid' }] });
parent.save(function (err) {
  console.log(err.message) // #sadpanda
});
```

子文档的pre（'save'）和pre（'validate'）中间件在顶层文档的pre（'save'）之前执行，但在顶层文档的pre（'validate'）中间件之后执行。因为 验证在 save() 之前执行, 也是内置中间件的一部分.

```js
// Below code will print out 1-4 in order
var childSchema = new mongoose.Schema({ name: 'string' });

childSchema.pre('validate', function(next) {
  console.log('2');
  next();
});

childSchema.pre('save', function(next) {
  console.log('3');
  next();
});

var parentSchema = new mongoose.Schema({
  child: childSchema,
    });

parentSchema.pre('validate', function(next) {
  console.log('1');
  next();
});

parentSchema.pre('save', function(next) {
  console.log('4');
  next();
});
```

## Finding a subdocument

每个子文档默认都有一个` _id`。 Mongoose文档数组有一个特殊的id方法用于搜索文档数组以找到具有给定 `_id` 的文档。

```js
var doc = parent.children.id(_id);
```

## 把子文档添加到数组

诸如push，unshift，addToSet和其他方法的MongooseArray方法透明地将参数转换为正确的类型：

```js
var Parent = mongoose.model('Parent');
var parent = new Parent;

// create a comment
parent.children.push({ name: 'Liesl' });
var subdoc = parent.children[0];
console.log(subdoc) // { _id: '501d86090d371bab2c0341c5', name: 'Liesl' }
subdoc.isNew; // true

parent.save(function (err) {
  if (err) return handleError(err)
  console.log('Success!');
});
```

不通过MongooseArrays 的 create 方法添加到数组, 子文档也可以被创建:

```js
var newdoc = parent.children.create({ name: 'Aaron' });
```

## 移除 子文档

每个子文档都有它自己的 remove 方法。对于数组子文档，这相当于在子文档上调用.pull（）。对于单个嵌套子文档，remove（）等同于将子文档设置为null。

```js
// Equivalent to `parent.children.pull(_id)`
parent.children.id(_id).remove();
// Equivalent to `parent.child = null`
parent.child.remove();
parent.save(function (err) {
  if (err) return handleError(err);
  console.log('the subdocs were removed');
});
```

**数组的替代声明语法**

如果您使用元素是对象的数组组创建schema，mongoose会自动将对象转换为schema：

```js
var parentSchema = new Schema({
  children: [{ name: 'string' }]
});
// Equivalent
var parentSchema = new Schema({
  children: [new Schema({ name: 'string' })]
});
```
