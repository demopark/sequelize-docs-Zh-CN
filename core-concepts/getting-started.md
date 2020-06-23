# Getting Started - 入门

在本教程中,你将进行学习 Sequelize 的简单设置.

## 安装

Sequelize 的使用可以通过 [npm](https://www.npmjs.com/package/sequelize) (或 [yarn](https://yarnpkg.com/package/sequelize)).

```sh
npm install --save sequelize
```

你必须手动为你的数据库安装驱动程序：

```sh
# 选择以下之一:
$ npm install --save pg pg-hstore # Postgres
$ npm install --save mysql2
$ npm install --save mariadb
$ npm install --save sqlite3
$ npm install --save tedious # Microsoft SQL Server
```

## 连接到数据库

要连接到数据库,必须创建一个 Sequelize 实例. 这可以通过将连接参数分别传递到 Sequelize 构造函数或传递一个数据库URL 来完成：

```js
const { Sequelize } = require('sequelize'); //引入sequelize核心包

// 方法 1: 传递一个连接 URI
const sequelize = new Sequelize('sqlite::memory:') // Sqlite 示例 
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname') // Postgres 示例

// 方法 2: 分别传递参数 (sqlite)
const sequelize = new Sequelize({
  dialect: 'sqlite', //数据库类型
  storage: 'path/to/database.sqlite'
});

// 方法 2: 分别传递参数 (其它数据库)
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* 选择 'mysql' | 'mariadb' | 'postgres' | 'mssql' 中的任何一种 */
});
```

Sequelize 构造函数接受很多参数. 它们的可传递参数在 [API 参考](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)中.

### 测试连接

你可以使用 `.authenticate()` 函数测试连接是否正常：

```js
try {
  await sequelize.authenticate();
  console.log('Connection has been established successfully.');
} catch (error) {
  console.error('Unable to connect to the database:', error);
}
```

### 关闭连接

默认情况下,Sequelize 将保持连接打开状态,并对所有查询使用相同的连接. 如果你需要关闭连接,请调用 `sequelize.close()`(这是异步的,并返回一个 Promise).

## 术语约定

请注意,在上面的示例中,`Sequelize` 是指库本身,而 `sequelize` 是指 Sequelize 的实例,它表示与一个数据库的连接. 这是官方推荐的约定,在整个文档中都将遵循.

## 阅读文档的提示

我们鼓励你在阅读 Sequelize 文档时在本地运行代码示例. 这将帮助你更快地学习. 最简单的方法是使用 SQLite 作为学习数据库：

```js
const { Sequelize, Op, Model, DataTypes } = require("sequelize");
const sequelize = new Sequelize("sqlite::memory:");

// 这是代码! 它是可用的!
```

要尝试使用在本地难以设置的其他方言,可以使用 [Sequelize SSCCE](https://github.com/papb/sequelize-sscce) GitHub 存储库,该库可让你在所有受支持的方言上运行代码, 直接从 GitHub 免费获得,无需任何设置！

## 新数据库与现有数据库

如果你是一个新建项目,并且没用创建任何相关的数据库,那么你可以使用 Sequelize 来帮助你自动创建数据库表.

除此之外,如果你想使用 Sequelize 连接到已经充满了表和数据的数据库,那也可以正常工作！ 在两种情况下,Sequelize 都能满足你的要求.

## 记录日志

默认情况下,Sequelize 会记录每一个SQL查询语句. 可以使用 `options.logging` 参数来自定义每次 Sequelize 记录某些内容时将执行的函数. 默认值为 `console.log`,使用该值时仅显示日志函数调用的第一个参数. 例如,对于查询日志记录,第一个参数是原始查询,第二个参数(默认情况下是隐藏的)是 Sequelize 对象.

`options.logging` 的常用值：

```js
const sequelize = new Sequelize('sqlite::memory:', {
  // 选择一种日志记录参数
  logging: console.log,                  // 默认值,显示日志函数调用的第一个参数
  logging: (...msg) => console.log(msg), // 显示所有日志函数调用参数
  logging: false,                        // 禁用日志记录
  logging: msg => logger.debug(msg),     // 使用自定义记录器(例如Winston 或 Bunyan),显示第一个参数
  logging: logger.debug.bind(logger)     // 使用自定义记录器的另一种方法,显示所有消息
});
```

## Bluebird Promises 和 async/await

Sequelize 提供的大多数方法都是异步的,因此返回 Promises. 它们都是 [Bluebird](http://bluebirdjs.com) Promises,因此你可以使用丰富的Bluebird API(例如,使用`finally`,`tap`,`tapCatch`,`map`,`mapSeries`, 等). 如果要设置任何特定的 Bluebird 的参数,则可以使用 `Sequelize.Promise` 访问 Sequelize 内部使用的 Bluebird 构造函数.

当然,使用 `async` 和 `await` 也可以正常工作.
