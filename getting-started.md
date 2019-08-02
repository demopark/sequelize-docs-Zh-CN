# Getting started - 入门

在本教程中,将通过简单地设置 Sequelize 来学习基础知识.

## 安装

Sequelize 可通过 [npm](https://www.npmjs.com/package/sequelize) ( 或 [yarn](https://yarnpkg.com/package/sequelize) ) 获得.

```sh
// 通过 npm 安装
npm install --save sequelize
```

你还需要手动安装对应的数据库驱动程序:

```sh
# 选择对应的安装:
$ npm install --save pg pg-hstore # Postgres
$ npm install --save mysql2
$ npm install --save mariadb
$ npm install --save sqlite3
$ npm install --save tedious # Microsoft SQL Server
```

## 建立连接

要连接到数据库,你必须创建 Sequelize 实例. 这可以通过将连接参数分别传递给 Sequelize 构造函数或传递单个连接 URI 来完成:

```js
const Sequelize = require('sequelize');

//方法1:单独传递参数
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* 'mysql' | 'mariadb' | 'postgres' | 'mssql' 之一 */
});

// 方法2: 传递连接 URI
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

Sequelize 构造函数采用了 [Sequelize构造函数的API参考](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor) 中记录的大量参数.

### 注意: 设置 SQLite

如果你正在使用 SQLite, 你应该使用以下代码:

```js
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});
```

### 注意: 连接池 (生产环境)

如果从单个进程连接到数据库,则应仅创建一个 Sequelize 实例. Sequelize 将在初始化时设置连接池. 可以通过构造函数的 `options` 参数(使用`options.pool`)配置此连接池,如以下示例所示:

```js
const sequelize = new Sequelize(/* ... */, {
  // ...
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});
```

在[Sequelize构造函数API参考](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)中了解更多信息. 如果从多个进程连接到数据库,则必须为每个进程创建一个实例,但每个实例应具有最大连接池大小,以便遵守总的最大大小.例如,如果你希望最大连接池大小为 90 并且你有三个进程,则每个进程的 Sequelize 实例的最大连接池大小应为 30.

## 测试连接

你可以使用  `.authenticate()` 函数来测试连接.

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

### 关闭连接

Sequelize 将默认保持连接持续,并对所有查询使用相同的连接. 如果需要关闭连接,请调用`sequelize.close()`(这是异步的并返回Promise).

## 表建模

模型是一个扩展 Sequelize.Model 的类. 模型可以用两种等效方式定义. 第一个是Sequelize.Model.init(属性,参数):

```js
const Model = Sequelize.Model;
class User extends Model {}
User.init({
  // 属性
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull 默认为 true
  }
}, {
  sequelize,
  modelName: 'user'
  // 参数
});
```

另一个是使用 `sequelize.define`:

```js
const User = sequelize.define('user', {
  // 属性
  firstName: {
    type: Sequelize.STRING,
    allowNull: false
  },
  lastName: {
    type: Sequelize.STRING
    // allowNull 默认为 true
  }
}, {
  // 参数
});
```

在内部, `sequelize.define` 调用 `Model.init`.

上面的代码告诉 Sequelize 在数据库中期望一个名为 `users` 的表,其中包含 `firstName` 和 `lastName` 字段. 默认情况下,表名自动复数(在当下使用[inflection](https://www.npmjs.com/package/inflection) 库来执行此操作).通过使用 `freezeTableName:true` 参数可以为特定模型停止此行为,或者通过使用[Sequelize构造函数](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)中的 `define` 参数为所有模型停止此行为.

Sequelize 还默认为每个模型定义了字段`id`(主键),`createdAt`和`updatedAt`. 当然也可以更改此行为(请查看API参考以了解有关可用参数的更多信息).

### 更改默认模型参数

Sequelize 构造函数采用 `define` 参数,它将更改所有已定义模型的默认参数.

```js
const sequelize = new Sequelize(connectionURI, {
  define: {
    // `timestamps` 字段指定是否将创建 `createdAt` 和 `updatedAt` 字段.
    // 该值默认为 true, 但是当前设定为 false
    timestamps: false
  }
});

// 这里 `timestamps` 为 false,因此不会创建 `createdAt` 和 `updatedAt` 字段.
class Foo extends Model {}
Foo.init({ /* ... */ }, { sequelize });

// 这里 `timestamps` 直接设置为 true,因此将创建 `createdAt` 和 `updatedAt` 字段.
class Bar extends Model {}
Bar.init({ /* ... */ }, { sequelize, timestamps: true });
```

你可以在[Model.init API 参考](/class/lib/model.js~Model.html#static-method-init) 或  [sequelize.define API 参考](/class/lib/sequelize.js~Sequelize.html#instance-method-define) 中阅读有关创建模型的更多信息.

## 将模型与数据库同步

如果你希望 Sequelize 根据你的模型定义自动创建表(或根据需要进行修改),你可以使用`sync`方法,如下所示:

```js
// 注意:如果表已经存在,使用`force:true`将删除该表
User.sync({ force: true }).then(() => {
  // 现在数据库中的 `users` 表对应于模型定义
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```

### 一次同步所有模型

你可以调用`sequelize.sync()`来自动同步所有模型,而不是为每个模型调用`sync()`.

### 生产环境注意事项

在生产环境中,你可能需要考虑使用迁移而不是在代码中调用`sync()`.阅读 [Migrations(迁移)](http://docs.sequelizejs.com/manual/migrations.html) 了解更多信息.

## 查询

一些简单的查询如下所示:

```js
// 查找所有用户
User.findAll().then(users => {
  console.log("All users:", JSON.stringify(users, null, 4));
});

// 创建新用户
User.create({ firstName: "Jane", lastName: "Doe" }).then(jane => {
  console.log("Jane's auto-generated ID:", jane.id);
});

// 删除所有名为“Jane”的人
User.destroy({
  where: {
    firstName: "Jane"
  }
}).then(() => {
  console.log("Done");
});

// 将所有没有姓氏的人改为“Doe”
User.update({ lastName: "Doe" }, {
  where: {
    lastName: null
  }
}).then(() => {
  console.log("Done");
});
```

Sequelize 有很多查询参数. 你将在下一个教程中了解有关这些内容的更多信息. 也可以进行原始SQL查询,如果你真的需要它们.

## Promises 和 async/await

如上所示,通常普遍使用 `.then` 调用,Sequelize 普遍使用 Promise. 这意味着,如果你的 Node 版本支持它,你可以对使用 Sequelize 进行的所有异步调用使用 ES2017 `async/await` 语法.

此外,所有 Sequelize promises 实际上都是 [Bluebird](http://bluebirdjs.com) promises,所以你也可以使用丰富的 Bluebird API(例如,使用`finally`,`tap`,`tapCatch`,`map`,`mapSeries`等). 如果要设置任何 Bluebird 特定参数,可以使用 Sequelize 内部使用的`Sequelize.Promise` 访问 Bluebird 构造函数.