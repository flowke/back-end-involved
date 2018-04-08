## 快速上手

## 连接到数据库

```js
const mongoose = require('mongoose');

// 进行连接
mongoose.connect('mongodb://localhost/test');

// 监控连接的状态, 成功后可以开始做其它操作
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  console.log('connected to mongodb');
});
```

## 创建一个 model

```js
let blogSchema = new mongoose.Schema({
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

let Blog = mongoose.model('Blog', blogSchema);
```

## 添加一条数据
