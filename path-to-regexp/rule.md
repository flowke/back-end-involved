# 路径规则

## 参数

#### 有名参数

假如路径规则: `/:foo/:bar`, 访问路径是: `/a/b`, 那么 `foo=>a, bar=>b`

> 参数名称必须由“单词字符”（[A-Za-z0-9_]）组成。

#### 参数修饰符

**可选修饰: ?**

参数可以带有 `?` 后缀表面参数可选:

`/:foo/:bar?` 表示 bar 这个参数有没有都行.

`/a/b` 或 `/a` 都能匹配.

> 如果参数是段中唯一的值，则前缀也是可选的。

**0或更多: \***

参数可以加后缀星号（*）来表示零个或多个参数匹配。每个匹配都会考虑前缀。

```js
var re = pathToRegexp('/:foo*')
// keys = [{ name: 'foo', delimiter: '/', optional: true, repeat: true }]

re.exec('/')
//=> ['/', undefined]

re.exec('/bar/baz')
//=> ['/bar/baz', 'bar/baz']
```

**1或更多: +**

参数可以加后缀星号（+）来表示零个或多个参数匹配。每个匹配都会考虑前缀。

```js

var re = pathToRegexp('/:foo+')
// keys = [{ name: 'foo', delimiter: '/', optional: false, repeat: true }]

re.exec('/')
//=> null

re.exec('/bar/baz')
//=> ['/bar/baz', 'bar/baz']
```
