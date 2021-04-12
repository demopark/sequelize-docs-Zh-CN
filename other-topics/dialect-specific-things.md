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

要通过 Unix 域套接字进行连接,请在 `host` 参数中指定套接字目录的路径. 套接字路径必须以 `/` 开头.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'postgres',
  host: '/path/to/socket_directory'
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