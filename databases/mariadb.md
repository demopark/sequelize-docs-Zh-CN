# Sequelize 用于 MariaDB



请参阅 [Releases](/releases#mariadb-support-table) 以了解支持哪些版本的 MariaDB。


要在 MariaDB 上使用 Sequelize，你需要安装 `@sequelize/mariadb` 方言包：

```bash npm2yarn
npm i @sequelize/mariadb
```


然后在 Sequelize 构造函数中使用 `MariaDbDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { MariaDbDialect } from '@sequelize/mariadb';

const sequelize = new Sequelize({
  dialect: MariaDbDialect,
  database: 'mydb',
  user: 'myuser',
  password: 'mypass',
  host: 'localhost',
  port: 3306,
  showWarnings: true,
  connectTimeout: 1000,
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


以下选项会原样传递给 Sequelize 用于连接 MariaDB 的 `mariadb` 包。
更多关于每个选项的作用，请参考 [MariaDB 官方文档](https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/b65aca10b77f5ede83f16a8edd0537b2ef12a16f/documentation/connection-options.md)。

为方便起见，下面是仅包含 Sequelize 支持选项的文档摘录：

| 选项                      | 说明                                                                                                                                                                                                                                                                                                                                                         |   类型            |   默认值           |
| :------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------: | :----------------: |
| `user`                    | 用于访问数据库的用户名                                                                                                                                                                                                                                                                                                                                       |      string       |                    |
| `password`                | 用户密码                                                                                                                                                                                                                                                                                                                                                    |      string       |                    |
| `host`                    | 数据库服务器的 IP 地址或 DNS。_使用 `socketPath` 选项时不适用_                                                                                                                                                                                                                                                        |      string       |    "localhost"     |
| `port`                    | 数据库服务器端口号                                                                                                                                                                                                                                                                                                    |      integer      |        3306        |
| `database`                | 建立连接时默认使用的数据库                                                                                                                                                                                                                                                                                             |      string       |                    |
| `socketPath`              | 允许通过 Unix 域套接字或命名管道连接数据库（如果服务器允许）                                                                                                                                                                                                                      |      string       |                    |
| `compress`                | 使用 gzip 压缩与数据库的通信。当访问远程数据库时可提升性能。                                                                                                                                                                                                                      |      boolean      |       false        |
| `connectTimeout`          | 连接超时时间（毫秒）                                                                                                                                                                                                                                                                                                  |      integer      |        1000        |
| `socketTimeout`           | 连接建立后，套接字超时时间（毫秒）                                                                                                                                                                                                                                                                                    |      integer      |         0          |
| `maxAllowedPacket`        | 可指示服务器全局变量 [max_allowed_packet](https://mariadb.com/kb/en/library/server-system-variables/#max_allowed_packet) 的值，以确保高效批处理。默认 4Mb。详见 [批处理文档]                                                                                                   |      integer      |      4196304       |
| `prepareCacheLength`      | Define prepare LRU cache length. 0 means no cache                                                                                                                                                                                                                                                                                                            |        int        |        256         |
| `ssl`                     | SSL options. See [SSL options in the MariaDB documentation]                                                                                                                                                                                                                                                                                                  | boolean \| object |       false        |
| `charset`                 | Protocol character set used with the server. Connection collation will be the [default collation] associated with charset. It's mainly used for micro-optimizations. The default is often sufficient.                                                                                                                                                        |      string       |      UTF8MB4       |
| `collation`               | (used in replacement of charset) Permit to defined collation used for connection. This will defined the charset encoding used for exchanges with database and defines the order used when comparing strings. It's mainly used for micro-optimizations                                                                                                        |      string       | UTF8MB4_UNICODE_CI |
| `debug`                   | Logs all exchanges with the server. Displays in hexa.                                                                                                                                                                                                                                                                                                        |      boolean      |       false        |
| `debugLen`                | String length of logged message / error or trace                                                                                                                                                                                                                                                                                                             |      integer      |        256         |
| `logParam`                | indicate if parameters must be logged by query logger.                                                                                                                                                                                                                                                                                                       |      boolean      |       false        |
| `foundRows`               | When enabled, the update number corresponds to update rows. When disabled, it indicates the real rows changed.                                                                                                                                                                                                                                               |      boolean      |        true        |
| `multipleStatements`      | Allows you to issue several SQL statements in a single `query()` call. (That is, `INSERT INTO a VALUES('b'); INSERT INTO c VALUES('d');`). <br/><br/>This may be a **security risk** as it allows for SQL Injection attacks.                                                                                                                                 |      boolean      |       false        |
| `permitLocalInfile`       | Allows the use of `LOAD DATA INFILE` statements.<br/><br/>Loading data from a file from the client may be a security issue, as a man-in-the-middle proxy server can change the actual file the server loads. Being able to execute a query on the client gives you access to files on the client.                                                            |      boolean      |       false        |
| `pipelining`              | Sends queries one by one without waiting on the results of the previous entry. For more information, see [Pipelining](                                                                                                                                                                                                                                       |      boolean      |        true        |
| `trace`                   | Adds the stack trace at the time of query creation to the error stack trace, making it easier to identify the part of the code that issued the query. Note: This feature is disabled by default due to the performance cost of stack creation. Only turn it on when you need to debug issues.                                                                |      boolean      |       false        |
| `connectAttributes`       | Sends information, (client name, version, operating system, Node.js version, and so on) to the [Performance Schema](https://mariadb.com/kb/en/library/performance-schema-session_connect_attrs-table/). When enabled, the Connector sends JSON attributes in addition to the defaults.                                                                       | boolean \| object |       false        |
| `sessionVariables`        | Permit to set session variables when connecting. Example: `sessionVariables: { idle_transaction_timeout: 10000 }`                                                                                                                                                                                                                                            |      object       |
| `initSql`                 | When a connection is established, permit to execute commands before using connection                                                                                                                                                                                                                                                                         |      string       |       array        |
| `bulk`                    | disabled bulk command in batch                                                                                                                                                                                                                                                                                                                               |      boolean      |
| `forceVersionCheck`       | Force server version check by explicitly using SELECT VERSION(), not relying on server initial packet.                                                                                                                                                                                                                                                       |      boolean      |       false        |
| `checkDuplicate`          | Indicate to throw an exception if result-set will not contain some data due to having duplicate identifier. <br/>JSON cannot have multiple identical key, so query like `SELECT 1 as i, 2 as i` cannot result in `{ i:1, i:2 }`, 'i:1' would be skipped. <br/>When `checkDuplicate` is enable (default) driver will throw an error if some data are skipped. |      boolean      |        true        |
| `keepAliveDelay`          | permit to enable socket keep alive, setting delay. 0 means not enabled. Keep in mind that this don't reset server [`@@wait_timeout`](https://mariadb.com/kb/en/library/server-system-variables/#wait_timeout) (use pool option idleTimeout for that). in ms                                                                                                  |      integer      |                    |
| `rsaPublicKey`            | Indicate path/content to MySQL server RSA public key.                                                                                                                                                                                                                                                                                                        |      string       |                    |
| `cachingRsaPublicKey`     | Indicate path/content to MySQL server caching RSA public key.                                                                                                                                                                                                                                                                                                |      string       |                    |
| `allowPublicKeyRetrieval` | Indicate that if `rsaPublicKey` or `cachingRsaPublicKey` public key are not provided, if client can ask server to send public key.                                                                                                                                                                                                                           |      boolean      |       false        |
| `stream`                  | permits to set a function with parameter to set stream                                                                                                                                                                                                                                                                                                       |     function      |                    |
| `metaEnumerable`          | make resultset meta property enumerable                                                                                                                                                                                                                                                                                                                      |      boolean      |       false        |
| `infileStreamFactory`     | When LOAD LOCAL command executed, permit to set a callback function of type `(filepath?: string) => stream.Readable`. Connector will then not send file from LOAD LOCAL, but Readable content. This can permit to set extra validation of file path for example.                                                                                             |     function      |                    |
| `logPackets`              | Debug option : permit to save last exchanged packet. Error messages will display those last exchanged packet.                                                                                                                                                                                                                                                |      boolean      |       false        |
| `debugCompress`           | This will print all incoming and outgoing compressed packets on stdout.                                                                                                                                                                                                                                                                                      |      boolean      |       false        |
| `queryTimeout`            | Command execution timeout                                                                                                                                                                                                                                                                                                                                    |      number       |                    |

## Other MariaDB Options

The following options are also available for MariaDB:

| Option         | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `showWarnings` | If `true`, warnings produced during the execution of a query will be sent to the `logging` callback. Default is `false`. |

[batch documentation]: https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/b65aca10b77f5ede83f16a8edd0537b2ef12a16f/documentation/batch.md
[SSL options in the MariaDB documentation]: https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/b65aca10b77f5ede83f16a8edd0537b2ef12a16f/documentation/connection-options.md#configuration
[default collation]: https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/master/lib/const/collations.js#L372
[Pipelining]: https://github.com/mariadb-corporation/mariadb-connector-nodejs/blob/b65aca10b77f5ede83f16a8edd0537b2ef12a16f/documentation/pipelining.md
