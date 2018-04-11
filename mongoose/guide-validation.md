# 数据验证

在看语法之前, 先看看这些规则:

- 验证 在 SchemaType 中定义
- 验证 是中间件。 Mongoose默认在每次 `save` 之前验证.
- 您可以使用 `doc.validate（callback）` 或 `doc.validateSync（）` 手动运行验证
- 验证器不在 `undefined` 的值上运行。唯一的例外是 `required` 验证器。
- validation 是异步递归的; 当您调用 `Model＃save` 时，子文档验证也会执行。如果发生错误，您的`Model＃save`回调会收到它
- 验证是可定制的


```js
var schema = new Schema({
      name: {
        type: String,
        required: true
      }
    });
    var Cat = db.model('Cat', schema);

    // This cat has no name :(
    var cat = new Cat();
    cat.save(function(error) {
      assert.equal(error.errors['name'].message,
        'Path `name` is required.');

      error = cat.validateSync();
      assert.equal(error.errors['name'].message,
        'Path `name` is required.');
    });

```

## 内置验证器

Mongoose有几个内置的验证器。

- 所有 SchemaTypes 都有内置的 `required` 验证器, `required`验证起使用 SchemaType 的 `checkRequired（）`函数来验证。
- 数字有 `min` 和 `max` 验证器。
- 字符串有 `enum`，`match`，`maxlength` 和 `minlength` 验证器。

上面的每个验证器链接都提供了有关如何启用它们并自定义错误消息的更多信息。

```js
var breakfastSchema = new Schema({
      eggs: {
        type: Number,
        min: [6, 'Too few eggs'],
        max: 12
      },
      bacon: {
        type: Number,
        required: [true, 'Why no bacon?']
      },
      drink: {
        type: String,
        enum: ['Coffee', 'Tea'],
        required: function() {
          return this.bacon > 3;
        }
      }
    });
    var Breakfast = db.model('Breakfast', breakfastSchema);

    var badBreakfast = new Breakfast({
      eggs: 2,
      bacon: 0,
      drink: 'Milk'
    });
    var error = badBreakfast.validateSync();
    assert.equal(error.errors['eggs'].message,
      'Too few eggs');
    assert.ok(!error.errors['bacon']);
    assert.equal(error.errors['drink'].message,
      '`Milk` is not a valid enum value for path `drink`.');

    badBreakfast.bacon = 5;
    badBreakfast.drink = null;

    error = badBreakfast.validateSync();
    assert.equal(error.errors['drink'].message, 'Path `drink` is required.');

    badBreakfast.bacon = null;
    error = badBreakfast.validateSync();
    assert.equal(error.errors['bacon'].message, 'Why no bacon?');
```

## unique 选项不是验证器

## ...待更新部分

## 验证的错误

验证失败后会返回错误, 包含一个 `error` 对象:  `ValidatorError` 错误对象。

每个 `ValidatorError` 都有 `kind` , `path` , `value` 和 `message` 属性。

ValidatorError也可能有一个 `reason` 属性。如果你自己在自定义验证器中抛出了错误, 这个属性也会包含你抛出的错误.
