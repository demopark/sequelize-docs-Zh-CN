## Basic usage - 基本用法

在开始之前，你首先必须创建一个 Sequelize 的实例。 像下面这样：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
});
```

这将会保存要传递的数据库凭据并提供所有进一步的方法。

此外，你还可以指定非默认的主机或端口：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
  host: "my.server.tld",
  port: 9821,
})
```

如果你没有密码：

```js
const sequelize = new Sequelize({
  database: 'db_name',
  username: 'username',
  password: null,
  dialect: 'mysql'
});
```

你也可以使用连接字符串：

```js
const sequelize = new Sequelize('mysql://user:pass@example.com:9821/db_name', {
  // 更多选项请看下一节
})
```

## 选项

除了主机和端口，Sequelize 还提供了一大堆选项。它们在这:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // 数据库的 sql 方言
  // 当前已支持: 'mysql', 'sqlite', 'postgres', 'mssql'
  dialect: 'mysql',
  
  // 自定义主机; 默认值: localhost
  host: 'my.server.tld',
 
  // 自定义端口; 默认值: 依据 dialect 默认
  port: 12345,
 
  // 自定义协议，默认值: 'tcp'
  // 仅限 postgres, 用于 Heroku
  protocol: null,
 
  // 禁用日志; 默认值: console.log
  logging: false,
  
  // 你还可以将任何方言选项传递到底层方言库
  // - 默认是空
  // - 当前支持: 'mysql', 'postgres', 'mssql'
  dialectOptions: {
    socketPath: '/Applications/MAMP/tmp/mysql/mysql.sock',
    supportBigNumbers: true,
    bigNumberStrings: true
  },
 
  // sqlite 的存储引擎
  // - 默认值 ':memory:'
  storage: 'path/to/database.sqlite',
 
  // 禁止将未定义的值插入为NULL
  // - 默认值: false
  omitNull: true,
 
  // 是否使用本地库的标志
  // 如果是 'pg' -- 设置为 true 将允许 SSL 支持
  // - 默认值: false
  native: true,
 
  // 指定在调用 sequelize.define 时使用的选项
  // 如下示例:
  //   define: { timestamps: false }
  // 这基本等同于:
  //   sequelize.define(name, attributes, { timestamps: false })
  // 没有必要像这样去定义每个模型的时间戳选项
  define: {
    underscored: false
    freezeTableName: false,
    charset: 'utf8',
    dialectOptions: {
      collate: 'utf8_general_ci'
    },
    timestamps: true
  },
 
  // 类似于同步：你可以定义始终强制同步模型
  sync: { force: true },
 
  // 用于数据库连接池的池配置
  pool: {
    max: 5,
    idle: 30000,
    acquire: 60000,
  },

  // 每个事务的隔离级别. 
  // 默认为 dialect 默认
  isolationLevel: Transaction.ISOLATION_LEVELS.REPEATABLE_READ
})
```

**提示:** 你可以通过传递一个方法为日志部分设置一个自定义方法。第一个参数是将被记录的字符串 。

## 读取复制

Sequelize 支持读取复制，即在要执行 SELECT 查询时可以连接到多个服务器。 当你读取复制时，你指定一个或多个服务器作为读取副本，一个服务器充当写入主机，它处理所有写入和更新，并将其传播到副本（请注意，实际的复制进程为 *不是* 由 Sequelize 处理，而应该在后端数据库中设置）。

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mysql',
  port: 3306
  replication: {
    read: [
      { host: '8.8.8.8', username: 'read-username', password: 'some-password' },
      { host: '9.9.9.9', username: 'another-username', password: null }
    ],
    write: { host: '1.1.1.1', username: 'write-username', password: 'any-password' }
  },
  pool: { // 如果要覆盖用于读写池的选项，可以在此处进行
    max: 20,
    idle: 30000
  },
})
```

如果你有适用于所有副本的常规设置，则不需要为每个实例单独提供它们。在上面的代码中，数据库名称和端口被传播到所有副本。对于用户和密码也是如此， 如果你把它们用于任何一个副本。每个副本都有以下选项：`host`，`port`，`username`，`password`，`database`。

Sequelize使用池来管理到副本的连接。内部 Sequelize 将维护用 `pool` 配置创建的两个池。

如果要修改这些，可以在实例化 Sequelize 时作为选项传递池，如上所示。

每个 `write` 或 `useMaster:true` 查询将使用写入池。 对于`SELECT`，将使用读取池。 只读副本使用基本的循环调度进行切换。

## 方言

随着 Sequelize `1.6.0` 的发布，库可以独立于特定的方言。这意味着您必须自己添加相应的连接器库到您的项目。

### MySQL

为了使 Sequelize 与 MySQL 完美结合，您需要安装 `mysql2@^1.0.0-rc.10` 或更高版本。 一旦完成，你可以这样使用它：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
})
```

**注意:** 您可以通过设置 `dialectOptions` 参数将选项直接传递给方言库. 查看 [Options][0]
获取例子 (目前只支持mysql).

### SQLite

对于 SQLite 兼容性，您将需要 `sqlite3 @〜3.0.0`。 配置 Sequelize 如下所示：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // 设置成 sqlite
  dialect: 'sqlite',
 
  // sqlite 的存储引擎
  // - default ':memory:'
  storage: 'path/to/database.sqlite'
})
```

或者您也可以使用连接字符串以及路径:

```js
const sequelize = new Sequelize('sqlite:/home/abs/path/dbname.db')
const sequelize = new Sequelize('sqlite:relativePath/dbname.db')
```

### PostgreSQL

PostgreSQL 的库是 `pg@^5.0.0 || ^6.0.0` 你只需要定义方言:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // 定义为 postgres
  dialect: 'postgres'
})
```

**注意:** `pg@^7.0.0` 当前不被支持.

### MSSQL

MSSQL 的库是 `tedious@^1.7.0` 你只需要定义方言:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mssql'
})
```

## 执行原始 SQL 查询

由于常常使用简单的方式来执行原始/已经准备好的SQL查询，所以可以使用“sequelize.query”函数。

这是它如何工作的:

```js
// 原始查询的参数
sequelize.query('your query', [, options])

// 简单的例子
sequelize.query("SELECT * FROM myTable").then(myTableRows => {
  console.log(myTableRows)
})

// 如果要返回 sequelize 实例，请使用模型选项。
// 这样，你可以轻松地将查询映射到预定义的sequelize模型，例如：
sequelize
  .query('SELECT * FROM projects', { model: Projects })
  .then(projects => {
    // 每个记录现在将映射到项目的模型。
    console.log(projects)
  })


// 选项是具有以下键的对象：
sequelize
  .query('SELECT 1', {
    // 用于记录查询的函数（或false）
    // 每个发送到服务器的SQL查询都会调用
    logging: console.log,

    // 如果 plain 是 TRUE ，则 sequelize 将只返回结果集的第一条记录。
    // 如果是 FALSE， 则是全部记录。
    plain: false,

    // 如果你没有查询的模型定义，请将其设置为true。
    raw: false,
    
    // 您正在执行的查询类型。 查询类型会影响结果在传回之前的格式。
    type: Sequelize.QueryTypes.SELECT
  })

// 注意第二个参数为null！
// 即使我们在这里声明一个被调用，raw: true 将取代并返回一个原始对象。
sequelize
  .query('SELECT * FROM projects', { raw: true })
  .then(projects => {
    console.log(projects)
  })
```

查询中的替换可以通过两种不同的方式完成：
使用命名参数（以`:`开头），或者由未命名的

使用的语法取决于传递给函数的替换选项：

* 如果一个数组被传递，`?` 将按照它们在数组中出现的顺序被替换
* 如果传递一个对象，`:key`将被该对象的键替换。如果包含在查询中的对象未找到对应的键，则会抛出异常，反之亦然。

```js
sequelize
  .query(
    'SELECT * FROM projects WHERE status = ?',
    { raw: true, replacements: ['active']
  )
  .then(projects => {
    console.log(projects)
  })

sequelize
  .query(
    'SELECT * FROM projects WHERE status = :status ',
    { raw: true, replacements: { status: 'active' } }
  )
  .then(projects => {
    console.log(projects)
  })
```

**注意一点:** 如果表的属性名称包含 " . "，则生成的对象将被嵌套：

```js
sequelize.query('select 1 as `foo.bar.baz`').then(rows => {
  console.log(JSON.stringify(rows))

  /*
    [{
      "foo": {
        "bar": {
          "baz": 1
        }
      }
    }]
  */
})
```



[0]: /docs/latest/usage#options
