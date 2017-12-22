# Getting started - 入门

## 安装

Sequelize 可通过 NPM 和 Yarn 获得。

```bash
// 使用 NPM
$ npm install --save sequelize

# 还有以下之一:
$ npm install --save pg@6 pg-hstore #pg@7 当前不支持
$ npm install --save mysql2
$ npm install --save sqlite3
$ npm install --save tedious // MSSQL

// 使用 Yarn
$ yarn add sequelize

# 还有以下之一:
$ yarn add pg pg-hstore
$ yarn add mysql2
$ yarn add sqlite3
$ yarn add tedious // MSSQL
```

## 建立连接

Sequelize将在初始化时设置连接池，所以如果从单个进程连接到数据库，你最好每个数据库只创建一个实例。 如果要从多个进程连接到数据库，则必须为每个进程创建一个实例，但每个实例应具有“最大连接池大小除以实例数”的最大连接池大小。 因此，如果您希望最大连接池大小为90，并且有3个工作进程，则每个进程的实例应具有30的最大连接池大小。

```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },

  // 仅限 SQLite
  storage: 'path/to/database.sqlite'
});

// 或者你可以简单地使用 uri 连接
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

Sequelize 构造函数可以通过 [API reference](/class/lib/sequelize.js~Sequelize.html) 获得一整套可用的参数。

## 测试连接

您可以使用  `.authenticate()` 函数来测试连接。

```js
sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```

## 你的第一个模型

模型使用 `sequelize.define('name', {attributes}, {options})` 来定义.

```js
const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING
  },
  lastName: {
    type: Sequelize.STRING
  }
});

// force: true 如果表已经存在，将会丢弃表
User.sync({force: true}).then(() => {
  // 表已创建
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```

您可以在 [Model API reference](/class/lib/model.js~Model.html) 中阅读更多关于创建模型的信息。

## 你的第一个查询

```js
User.findAll().then(users => {
  console.log(users)
})
```

您可以在  [Data retrieval](/manual/tutorial/models-usage.html#data-retrieval-finders)  上查看更多关于模型的查找器功能,如 `.findAll()` 。或者在 [Querying](/manual/tutorial/querying.html) 上查看如何执行特定查询，如 `WHERE` 和 `JSONB` 。

### 应用全局的模型参数

Sequelize 构造函数使用 `define` 参数，该参数将用作所有定义模型的默认参数。

```js
const sequelize = new Sequelize('connectionUri', {
  define: {
    timestamps: false // 默认为 true
  }
});

const User = sequelize.define('user', {}); // 时间戳默认为 false
const Post = sequelize.define('post', {}, {
  timestamps: true // 时间戳此时为 false
});
```

## Promise

Sequelize 使用 [Bluebird](http://bluebirdjs.com) promise 来控制异步控制流程。

**注意:** _Sequelize 使用 Bluebird 实例的独立副本。如果你想设置任何 Bluebird 特定的参数可以通过使用 `Sequelize.Promise` 来访问它。_

如果你不熟悉 promise 是如何工作的，别担心，你可以阅读一下 [这里](http://bluebirdjs.com/docs/why-promises.html)。

基本上，一个 promise 代表了某个时候会出现的值 - 这意味着“我保证你会在某个时候给你一个结果或一个错误”。 

```js
// 不要这样做
user = User.findOne()

console.log(user.get('firstName'));
```

_这将永远不可用！_这是因为`user`是 promise 对象，而不是数据库中的数据行。 正确的方法是：

```js
User.findOne().then(user => {
  console.log(user.get('firstName'));
});
```

当您的环境或解释器支持 [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) 时，这将可用，但只能在  [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)  方法体中：

```js
user = await User.findOne()

console.log(user.get('firstName'));
```

一旦知道了什么是 promise 以及它们的工作原理，请使用 [bluebird API reference](http://bluebirdjs.com/docs/api-reference.html) 作为转移工具。 尤其是，你可能会使用很多 [`.all`](http://bluebirdjs.com/docs/api/promise.all.html) 。
