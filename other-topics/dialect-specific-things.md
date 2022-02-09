# Dialect-Specific Things - 方言特定事项

## 基础连接器库

### MySQL

Sequelize 对于 MySQL 使用的基础连接器库是 [mysql2](https://www.npmjs.com/package/mysql2) npm 软件包(1.5.2 或更高版本).

你可以使用 Sequelize 构造函数中的 `dialectOptions` 为其提供自定义参数：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql',
  dialectOptions: {
    // 你的 mysql2 参数
  }
})
```

`dialectOptions` 直接传递给 MySQL 连接构造函数. 完整的选项列表可以在 [MySQL 文档](https://www.npmjs.com/package/mysql#connection-options) 中找到.

### MariaDB

Sequelize 对于 MariaDB 使用的基础连接器库是 [mariadb](https://www.npmjs.com/package/mariadb) npm 软件包.

你可以使用 Sequelize 构造函数中的 `dialectOptions` 为其提供自定义参数：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mariadb',
  dialectOptions: {
    // 你的 mariadb 参数
    // connectTimeout: 1000
  }
});
```

`dialectOptions` 直接传递给 MariaDB 连接构造函数. 完整的选项列表可以在 [MariaDB 文档](https://mariadb.com/kb/en/nodejs-connection-options/) 中找到.

### SQLite

Sequelize 对于 SQLite 使用的基础连接器库是 [sqlite3](https://www.npmjs.com/package/sqlite3) npm 程序包(版本4.0.0或更高版本).

你可以在 Sequelize 构造函数中使用 `storage` 参数指定存储文件(对于内存中的SQLite实例,请使用 `:memory:`).

你可以使用 Sequelize 构造函数中的 `dialectOptions` 为其提供自定义参数：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite' // 或 ':memory:'
  dialectOptions: {
    // 你的 sqlite3 参数
  }
});
```

以下字段可以传递给 SQLite `dialectOptions`:

- `readWriteMode`: 设置 SQLite 连接的打开模式. 潜在值由 sqlite3 包提供, 并且能包括 sqlite3.OPEN_READONLY, sqlite3.OPEN_READWRITE 或 sqlite3.OPEN_CREATE. 查阅 [SQLite C 接口文档]( https://www.sqlite.org/c3ref/open.html) 以获取更多详细信息.

### PostgreSQL

Sequelize 对于 PostgreSQL 使用的基础连接器库是 [pg](https://www.npmjs.com/package/pg) npm 软件包(版本7.0.0或更高版本). 还需要模块 [pg-hstore](https://www.npmjs.com/package/pg-hstore).

你可以使用 Sequelize 构造函数中的 `dialectOptions` 为其提供自定义参数：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'postgres',
  dialectOptions: {
    // 你的 pg 参数
  }
});
```

以下字段可以传递给 Postgres `dialectOptions`:

- `application_name`: pg_stat_activity 中的应用程序名称. 参阅 [Postgres 文档](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-APPLICATION-NAME) 获取更多详细信息.
- `ssl`: SSL 参数. 参阅 [`pg` 文档](https://node-postgres.com/features/ssl) 获取更多详细信息.
- `client_encoding`: // 设置 'auto' 根据客户端 LC_CTYPE 环境变量确定语言环境. 参阅 [Postgres 文档](https://www.postgresql.org/docs/current/multibyte.html) 获取更多详细信息.
- `keepAlive`: 启用 TCP KeepAlive 的布尔值. 参阅 [`pg` 更新记录](https://github.com/brianc/node-postgres/blob/master/CHANGELOG.md#v600) 获取更多详细信息.
- `statement_timeout`: 在设定的时间后超时查询(以毫秒为单位). 添加于 pg v7.3. 参阅 [Postgres 文档](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT) 获取更多详细信息.
- `idle_in_transaction_session_timeout`: 终止超过指定持续时间(以毫秒为单位)的空闲事务会话. 参阅 [Postgres 文档](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-IDLE-IN-TRANSACTION-SESSION-TIMEOUT) 获取更多详细信息.

要通过 Unix 域套接字进行连接,请在 `host` 参数中指定套接字目录的路径. 套接字路径必须以 `/` 开头.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'postgres',
  host: '/path/to/socket_directory'
});
```

sequelize 中默认的 `client_min_messages` 配置是 `WARNING`.

### Redshift

大多数配置与上面的 PostgreSQL 相同.

Redshift 不支持 `client_min_messages`, 需要 'ignore' 跳过配置:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'postgres',
  dialectOptions: {
    // 你的 pg 参数
    // ...
    clientMinMessages: 'ignore' // 不区分大小写
  }
});
```

### MSSQL

Sequelize 用于 MSSQL 的基础连接器库是 [tedious](https://www.npmjs.com/package/tedious) npm 软件包(版本6.0.0或更高版本).

你可以使用 Sequelize 构造函数中的 `dialectOptions.options` 为其提供自定义参数：

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mssql',
  dialectOptions: {
    // 观察 MSSQL 这个嵌套的 `options` 字段
    options: {
      // 你的 tedious 参数
      useUTC: false,
      dateFirst: 1
    }
  }
});
```

完整的选项列表可以在 [tedious 文档](https://tediousjs.github.io/tedious/api-connection.html#function_newConnection) 中找到.

#### MSSQL 域账户

为了连接域帐户,请使用以下格式.

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mssql',
  dialectOptions: {
    authentication: {
      type: 'ntlm',
      options: {
        domain: 'yourDomain',
        userName: 'username',
        password: 'password'
      }
    },
    options: {
      instanceName: 'SQLEXPRESS'
    }
  }
})
```

### Snowflake (实验性)

Sequelize 用于 Snowflake 的底层连接器库是 [snowflake-sdk](https://www.npmjs.com/package/snowflake-sdk) npm 包.

为了与帐户连接, 请使用以下格式:

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'snowflake',
  dialectOptions: {
    // 把你的snowflake帐户放在这里,
    account: 'myAccount',  // my-app.us-east-1

    // 下面的选项是可选的
    role: 'myRole',
    warehouse: 'myWarehouse',
    schema: 'mySchema'
  },
  // 和其他方言一样
  username: 'myUserName',
  password: 'myPassword',
  database: 'myDatabaseName'
})
```

**注意** 没有提供测试沙箱, 因此 snowflake 集成测试不是 pipeline 的一部分. 核心团队也很难进行分类和调试. 这种方言现在需要由 snowflake 用户/社区维护.

用于运行集成测试:

```sh
SEQ_ACCOUNT=myAccount SEQ_USER=myUser SEQ_PW=myPassword SEQ_ROLE=myRole SEQ_DB=myDatabaseName SEQ_SCHEMA=mySchema SEQ_WH=myWareHouse npm run test-integration-snowflake
```

## 数据类型: TIMESTAMP WITHOUT TIME ZONE - 仅限  PostgreSQL

如果你使用的是 PostgreSQL `TIMESTAMP WITH TIME ZONE`,并且需要将其解析为其他时区,请使用 pg 库自己的解析器：

```js
require('pg').types.setTypeParser(1114, stringValue => {
  return new Date(stringValue + '+0000');
  // 例如 UTC 偏移量. 使用任何你想要的偏移量.
});
```

## 数据类型: ARRAY(ENUM) - 仅限 PostgreSQL

Array(Enum)类型需要特殊处理. 每当 Sequelize 与数据库对话时,它都必须使用 ENUM 名称转换数组值.

因此,此枚举名称必须遵循这种格式 `enum_<table_name>_<col_name>`. 如果你使用 `sync`,则将自动生成正确的名称.

## 表提示 - 仅限 MSSQL

`tableHint` 属性可用于定义表提示. 提示必须是来自 `TableHints` 的值,并且仅在绝对必要时才使用. 当前每个查询仅支持单个表提示.

表提示通过指定某些参数来覆盖 MSSQL 查询优化器的默认行为. 它们仅影响该子句中引用的表或视图.

```js
const { TableHints } = require('sequelize');
Project.findAll({
  // 添加表提示 NOLOCK
  tableHint: TableHints.NOLOCK
  // 这将生成 SQL 'WITH (NOLOCK)'
})
```

## 索引提示 - 仅限 MySQL/MariaDB

`indexHints` 参数可以用来定义索引提示. 提示类型必须是 `IndexHints` 中的值,并且这些值应引用现有索引.

索引提示[将覆盖 MySQL 查询优化器的默认行为](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html).

```js
const { IndexHints } = require("sequelize");
Project.findAll({
  indexHints: [
    { type: IndexHints.USE, values: ['index_project_on_name'] }
  ],
  where: {
    id: {
      [Op.gt]: 623
    },
    name: {
      [Op.like]: 'Foo %'
    }
  }
});
```

上面的代码将生成一个如下所示的 MySQL 查询：

```sql
SELECT * FROM Project USE INDEX (index_project_on_name) WHERE name LIKE 'FOO %' AND id > 623;
```

`Sequelize.IndexHints` 包含 `USE`, `FORCE`, 和 `IGNORE`.

参考 [Issue #9421](https://github.com/sequelize/sequelize/issues/9421) 有关原始 API 提案.

## 引擎 - 仅限 MySQL/MariaDB

模型的默认引擎是 InnoDB.

你可以使用 `engine` 参数更改模型的引擎(例如,更改为 MyISAM)：

```js
const Person = sequelize.define('person', { /* 属性 */ }, {
  engine: 'MYISAM'
});
```

像模型定义的每个参数一样,也可以使用 Sequelize 构造函数的 `define` 参数来全局更改此设置：

```js
const sequelize = new Sequelize(db, user, pw, {
  define: { engine: 'MYISAM' }
})
```

## 表注释 - 仅限 MySQL/MariaDB/PostgreSQL

你可以在定义模型时为表指定注释：

```js
class Person extends Model {}
Person.init({ /* 属性 */ }, {
  comment: "I'm a table comment!",
  sequelize
})
```

调用 `sync()` 时将设置注释.