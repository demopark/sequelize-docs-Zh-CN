# Dialect-Specific Things - 方言特定事项

## 底层连接器库

### PostgreSQL

Sequelize for PostgreSQL 使用的底层连接器库是 [pg](https://www.npmjs.com/package/pg) 包.查看 [Releases](https://sequelize.org/releases/#postgresql-support-table) 查看支持哪些版本的 PostgreSQL 和 pg. 

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

### Amazon Redshift

**注意**

虽然 Redshift 基于 PostgreSQL, 但它不支持与 PostgreSQL 相同的功能集. 
我们的 PostgreSQL 实施未针对 Redshift 进行集成测试, 并且支持有限.

大多数配置与上面的 PostgreSQL 相同.

Redshift 不支持 `client_min_messages`, 你必须要设置 'ignore' 来跳过配置:

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

### MariaDB

Sequelize 对于 MariaDB 使用的基础连接器库是 [mariadb](https://www.npmjs.com/package/mariadb) 软件包.请参阅 [Releases](https://sequelize.org/releases/#mariadb-support-table) 查看支持哪些版本的 MariaDB 和 mariadb (npm).

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

### MySQL

Sequelize 对于 MySQL 使用的基础连接器库是 [mysql2](https://www.npmjs.com/package/mysql2) 软件包.查看 [Releases](https://sequelize.org/releases/#mysql-support-table) 查看支持哪些版本的 MySQL 和 mysql2.

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

### Microsoft SQL Server (mssql)

Sequelize for MSSQL 使用的底层连接器库是 [tedious](https://www.npmjs.com/package/tedious) 包. 请参阅 [Releases](https://sequelize.org/releases/#microsoft-sql-server-mssql-support-table) 查看支持哪些版本的 SQL Server & tedious.

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

### SQLite

Sequelize 对于 SQLite 使用的基础连接器库是 [sqlite3](https://www.npmjs.com/package/sqlite3) 程序包.请参阅 [Releases](https://sequelize.org/releases/#sqlite-support-table) 查看支持哪些版本的 sqlite3.

你可以在 Sequelize 构造函数中使用 `storage` 参数指定存储文件(对于内存中的SQLite实例,请使用 `:memory:`).

你可以使用 Sequelize 构造函数中的 `dialectOptions` 为其提供自定义参数：

```js
import { Sequelize } from 'sequelize';
import SQLite from 'sqlite3';

const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite', // 或 ':memory:'
  dialectOptions: {
    // 你的 sqlite3 参数
    // 对于实例, 这是配置数据库打开模式的方法:
    mode: SQLite.OPEN_READWRITE | SQLite.OPEN_CREATE | SQLite.OPEN_FULLMUTEX,
  }
});
```

以下字段可以传递给 SQLite `dialectOptions`:

- `mode`: 设置 SQLite 连接的打开模式. 潜在值由 `sqlite3` 包提供, 并且包括 `SQLite.OPEN_READONLY`, `SQLite.OPEN_READWRITE`, 或 `SQLite.OPEN_CREATE`. 请参阅 [sqlite3 的 API 参考](https://github.com/TryGhost/node-sqlite3/wiki/API) 和 [SQLite C 接口文档](https://www.sqlite.org/c3ref/open.html) 获取更多细节.

### Snowflake

**注意**

虽然这种方言包含在 Sequelize 中, 但是对 Snowflake 的支持是有限的, 因为它不是由核心团队处理的.

Sequelize 用于 Snowflake 的底层连接器库是 [snowflake-sdk](https://www.npmjs.com/package/snowflake-sdk) 包.请参阅 [Releases](https://sequelize.org/releases/#snowflake-support-table) 查看支持哪些版本的 Snowflake 和 snowflake-sdk.

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
});
```

**注意** 没有提供测试沙箱, 因此 snowflake 集成测试不是 pipeline 的一部分. 核心团队也很难进行分类和调试. 这种方言现在需要由 snowflake 用户/社区维护.

用于运行集成测试:

```bash
# using npm
SEQ_ACCOUNT=myAccount SEQ_USER=myUser SEQ_PW=myPassword SEQ_ROLE=myRole SEQ_DB=myDatabaseName SEQ_SCHEMA=mySchema SEQ_WH=myWareHouse npm run test-integration-snowflake
# using yarn
SEQ_ACCOUNT=myAccount SEQ_USER=myUser SEQ_PW=myPassword SEQ_ROLE=myRole SEQ_DB=myDatabaseName SEQ_SCHEMA=mySchema SEQ_WH=myWareHouse yarn test-integration-snowflake
```

### Db2

**注意**

虽然这种方言包含在 Sequelize 中, 但对 Db2 的支持是有限的, 因为它不是由核心团队处理的.

Sequelize for Db2 使用的底层连接器库是 [ibm_db](https://www.npmjs.com/package/ibm_db) npm 包.
请参阅 [Releases](https://sequelize.org/releases/#db2-support-table) 以查看支持哪些版本的 DB2 和 ibm_db.

### Db2 for IBM i

**注意**

虽然这种方言包含在 Sequelize 中, 但对 *Db2 for IBM i* 的支持是有限的, 因为它不是由核心团队处理的.

Sequelize 为 *Db2 for IBM i* 使用的底层连接器库是 [odbc](https://www.npmjs.com/package/odbc) npm 包.
请参阅 [Releases](https://sequelize.org/releases/#db2-for-ibm-i-support-table) 查看支持哪些版本的 IBMi 和 odbc.

要了解有关将 ODBC 与 IBM i 结合使用的更多信息, 请参阅 [IBM i 和 ODBC 文档](https://ibmi-oss-docs.readthedocs.io/en/latest/odbc/README.html).

将参数传递给构造函数时, `database` 的概念被映射到 ODBC 的 `DSN`. 你可以使用  `dialectOptions.odbcConnectionString` 为 Sequelize 提供额外的连接字符串参数. 然后, 此连接字符串会附加在参数中找到 `database`, `username`, 和 `password` 的值:

```js
const sequelize = new Sequelize('MY_DSN', 'username', 'password', {
  dialect: 'ibmi',
  dialectOptions: {
    odbcConnectionString: 'CMT=1;NAM=0;...'
  },
});
```

上述配置生成的最终连接字符串类似于`CMT=1;NAMING=0;...;DSN=MY_DSN;UID=username;PWD=password;`. 此外, `host` 参数将映射 `SYSTEM=` 连接字符串键.

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
const { TableHints } = require('@sequelize/core');
Project.findAll({
  // 添加表提示 NOLOCK
  tableHint: TableHints.NOLOCK
  // 这将生成 SQL 'WITH (NOLOCK)'
});
```

## 索引提示 - 仅限 MySQL/MariaDB

`indexHints` 参数可以用来定义索引提示. 提示类型必须是 `IndexHints` 中的值,并且这些值应引用现有索引.

索引提示[将覆盖 MySQL 查询优化器的默认行为](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html).

```js
const { IndexHints } = require('@sequelize/core');
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
});
```

## 表注释 - 仅限 MySQL/MariaDB/PostgreSQL

你可以在定义模型时为表指定注释：

```js
class Person extends Model {}
Person.init({ /* 属性 */ }, {
  comment: "I'm a table comment!",
  sequelize
});
```

调用 `sync()` 时将设置注释.