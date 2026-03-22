# Sequelize 用于 MySQL



请参阅 [Releases](/releases#mysql-support-table) 以了解支持哪些版本的 MySQL。


要在 MySQL 上使用 Sequelize，你需要安装 `@sequelize/mysql` 方言包：

```bash npm2yarn
npm i @sequelize/mysql
```


然后在 Sequelize 构造函数中使用 `MySqlDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { MySqlDialect } from '@sequelize/mysql';

const sequelize = new Sequelize({
  dialect: MySqlDialect,
  database: 'mydb',
  user: 'myuser',
  password: 'mypass',
  host: 'localhost',
  port: 3306,
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


以下选项会原样传递给 Sequelize 用于连接 MySQL 的 `mysql2` 包。
更多关于每个选项的作用，请参考 [mysql2 官方文档](https://sidorares.github.io/node-mysql2/docs)。

为方便起见，下面是仅包含 Sequelize 支持选项的文档摘录：

| 选项                    | 说明                                                                                                                                                                                                                                                            |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `database`              | 此连接要使用的数据库名称                                                                                                                                                                                                                                         |
| `user`                  | 用于认证的 MySQL 用户名                                                                                                                                                                                                                                         |
| `port`                  | 要连接的端口号（默认：3306）                                                                                                                                                                                                                                   |
| `host`                  | 要连接的数据库主机名（默认：localhost）                                                                                                                                                                                                                        |
| `localAddress`          | 用于 TCP 连接的源 IP 地址                                                                                                                                                                                                                                      |
| `password`              | 该 MySQL 用户的密码                                                                                                                                                                                                                                            |
| `password1`             | MySQL 用户密码的别名。在多因素认证场景下更有意义（参见 "password2" 和 "password3"）                                                                                                                                      |
| `password2`             | 第二因素认证密码。当 MySQL 用户账户的认证策略要求额外的密码认证方法时必填。详见：https://dev.mysql.com/doc/refman/8.0/en/multifactor-authentication.html                                                        |
| `password3`             | 第三因素认证密码。当 MySQL 用户账户的认证策略要求两个额外的认证方法且最后一个需要密码时必填。详见：https://dev.mysql.com/doc/refman/8.0/en/multifactor-authentication.html                                 |
| `passwordSha1`          | _暂无文档说明_                                                                                                                                                                                                                                                 |
| `socketPath`            | 用于连接的 unix 域套接字路径。使用该选项时 host 和 port 会被忽略。                                                                                                                                                      |
| `ssl`                   | 包含 ssl 参数的对象或包含 ssl 配置名称的字符串                                                                                                                                                                           |
| `charset`               | The charset for the connection. This is called 'collation' in the SQL-level of MySQL (like utf8_general_ci). If a SQL-level charset is specified (like utf8mb4) then the default collation for that charset is used. (Default: 'UTF8_GENERAL_CI')                 |
| `compress`              | _no documentation available_                                                                                                                                                                                                                                      |
| `trace`                 | Generates stack traces on Error to include call site of library entrance ('long stack traces'). Slight performance penalty for most calls. (Default: true)                                                                                                        |
| `enableKeepAlive`       | Enable keep-alive on the socket. (Default: true)                                                                                                                                                                                                                  |
| `isServer`              | _no documentation available_                                                                                                                                                                                                                                      |
| `insecureAuth`          | Allow connecting to MySQL instances that ask for the old (insecure) authentication method. (Default: false)                                                                                                                                                       |
| `multipleStatements`    | Allow multiple mysql statements per query. Be careful with this, it exposes you to SQL injection attacks. (Default: false)                                                                                                                                        |
| `waitForConnections`    | _no documentation available_                                                                                                                                                                                                                                      |
| `connectionLimit`       | _no documentation available_                                                                                                                                                                                                                                      |
| `connectTimeout`        | The milliseconds before a timeout occurs during the initial connection to the MySQL server. (Default: 10 seconds)                                                                                                                                                 |
| `charsetNumber`         | _no documentation available_                                                                                                                                                                                                                                      |
| `maxIdle`               | _no documentation available_                                                                                                                                                                                                                                      |
| `queueLimit`            | _no documentation available_                                                                                                                                                                                                                                      |
| `idleTimeout`           | _no documentation available_                                                                                                                                                                                                                                      |
| `maxPreparedStatements` | _no documentation available_                                                                                                                                                                                                                                      |
| `keepAliveInitialDelay` | If keep-alive is enabled users can supply an initial delay. (Default: 0)                                                                                                                                                                                          |
| `infileStreamFactory`   | By specifying a function that returns a readable stream, an arbitrary stream can be sent when sending a local fs file.                                                                                                                                            |
| `flags`                 | List of connection flags to use other than the default ones. It is also possible to denylist default ones                                                                                                                                                         |
| `authSwitchHandler`     | _no documentation available_                                                                                                                                                                                                                                      |
| `connectAttributes`     | _no documentation available_                                                                                                                                                                                                                                      |
| `authPlugins`           | _no documentation available_                                                                                                                                                                                                                                      |
| `debug`                 | This will print all incoming and outgoing packets on stdout. You can also restrict debugging to packet types by passing an array of types (strings) to debug;                                                                                                     |
| `stream`                | _no documentation available_                                                                                                                                                                                                                                      |

## Other MySQL Options

The following options are also available for MySQL:

| Option         | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `showWarnings` | If `true`, warnings produced during the execution of a query will be sent to the `logging` callback. Default is `false`. |
