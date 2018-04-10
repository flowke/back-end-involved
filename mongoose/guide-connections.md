# Connections

可以使用 `mongoose.connect（）`方法连接到MongoDB。

用于开启默认mongoose连接。

```js
mongoose.connect('mongodb://localhost/myapp');
```

#### 参数
```js
mongoose.connect(url, options, callback);
```

**url:**

举例: `mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]`

- `mongodb://`: 所需的前缀，用于表明这是标准连接格式。
- `username:password@`: 可选的。如果指定，则在连接到mongod实例后，客户端将尝试使用这些凭据登录到特定数据库。
- `host1`: 需要。它标识要连接的服务器地址。它标识主机名，IP地址或UNIX域套接字。
- `:port1`: 可选的。如果未指定，则默认值为：27017。
- `/database`: 可选的。要验证的数据库. 使用前面写的 username:password@ 来验证. 如果未指定/ database，并且连接字符串包含凭据，则驱动程序将向管理数据库进行身份验证。
- ?options

##...待更新
