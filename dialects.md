# Dialects - 方言

Sequelize 独立于特定方言. 这意味着你必须自己将相应的连接器库安装到项目中.

## MySQL

为了让 Sequelize 与 MySQL 一起更好地工作,你需要安装 `mysql2@^1.5.2` 或更高版本. 一旦完成,你可以像这样使用它:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mysql'
})
```

**注意:** 你可以通过设置 `dialectOptions` 参数将选项直接传递给方言库.

## MariaDB

MariaDB 的库是 `mariadb`.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mariadb',
  dialectOptions: {connectTimeout: 1000} // mariadb 连接参数
})
```

或使用连接字符串:

```js
const sequelize = new Sequelize('mariadb://user:password@example.com:9821/database')
```

## SQLite

由于 SQLite 兼容性,你需要`sqlite3@^4.0.0`. 像这样配置 Sequelize:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // sqlite!
  dialect: 'sqlite',

  // sqlite 的存储引擎
  // - default ':memory:'
  storage: 'path/to/database.sqlite'
})
```

或者你也可以使用连接字符串和路径:

```js
const sequelize = new Sequelize('sqlite:/home/abs/path/dbname.db')
const sequelize = new Sequelize('sqlite:relativePath/dbname.db')
```

## PostgreSQL

对于 PostgreSQL,需要两个库,`pg@^7.0.0` 和 `pg-hstore`. 你只需要定义方言:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // postgres!
  dialect: 'postgres'
})
```

要通过 unix 域套接字进行连接,请在 `host` 选项中指定套接字目录的路径.

套接字路径必须以 `/` 开头.

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  // postgres!
  dialect: 'postgres',
  host: '/path/to/socket_directory'
})
```

## MSSQL

MSSQL的库是 `tedious@^6.0.0` 你只需要定义方言:

```js
const sequelize = new Sequelize('database', 'username', 'password', {
  dialect: 'mssql'
})
```
