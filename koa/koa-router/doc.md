# koa-router

## 获取 url 参数

命名的路由参数被捕获并添加到ctx.params。

```js
router.get('/:category/:title', (ctx, next) => {
  console.log(ctx.params);
  // => { category: 'programming', title: 'how-to-node' }
});
```
