# Sequelize 用于 Microsoft SQL Server



请参阅 [Releases](/releases#microsoft-sql-server-mssql-support-table) 以了解支持哪些版本的 SQL Server。


要在 Microsoft SQL Server 上使用 Sequelize，你需要安装 `@sequelize/mssql` 方言包：

```bash npm2yarn
npm i @sequelize/mssql
```


然后在 Sequelize 实例中使用 `MsSqlDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { MsSqlDialect } from '@sequelize/mssql';

const sequelize = new Sequelize({
  dialect: MsSqlDialect,
  server: 'localhost',
  port: 1433,
  database: 'database',
  authentication: {
    type: 'default',
    options: {
      userName: 'username',
      password: 'password',
    },
  },
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


以下选项会原样传递给 Sequelize 用于连接 SQL Server 的 `tedious` 包。
更多关于每个选项的作用，请参考 [Tedious 官方文档](https://tediousjs.github.io/tedious/api-connection.html#function_newConnection)。

为方便起见，下面是仅包含 Sequelize 支持选项的文档摘录：

| 选项                          | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `server`                      | 要连接的主机名。                                                                                                                                                                                                                                                                                                                                                                                                       |
| `localAddress`                | 连接 SQL Server 时使用的网络接口（IP 地址）。                                                                                                                                                                                                                                                                                                                                                                         |
| `database`                    | 要连接的数据库。                                                                                                                                                                                                                                                                                                                                                                                                       |
| `port`                        | 要连接的端口（默认：1433）。<br/>与 `instanceName` 互斥。                                                                                                                                                                                                                                                                                                                                                             |
| `instanceName`                | 要连接的实例名。数据库服务器上必须运行 SQL Server Browser 服务，并且必须能访问数据库服务器的 UDP 1434 端口。<br/>与 `port` 互斥。                                                                                                                                                                                                                                               |
| `authentication`              | 认证选项。可用的子选项请参见 [Tedious 官方文档]。                                                                                                                                                                                                                                                                                                                             |
| `abortTransactionOnError`     | 布尔值，决定在事务执行期间遇到任何错误时是否自动回滚事务。此选项会在连接的初始 SQL 阶段设置 SET XACT_ABORT 的值（[文档](https://msdn.microsoft.com/en-us/library/ms188792.aspx)）。                                                                                                                                            |
| `appName`                     | Application name used for identifying a specific application in profiling, logging or tracing tools of SQL Server. (default: Tedious)                                                                                                                                                                                                                                                                                                                                                               |
| `cancelTimeout`               | The number of milliseconds before the cancel (abort) of a request is considered failed (default: 5000).                                                                                                                                                                                                                                                                                                                                                                                             |
| `connectionRetryInterval`     | Number of milliseconds before retrying to establish connection, in case of transient failure. (default: 500)                                                                                                                                                                                                                                                                                                                                                                                        |
| `connectTimeout`              | The number of milliseconds before the attempt to connect is considered failed (default: 15000).                                                                                                                                                                                                                                                                                                                                                                                                     |
| `connectionIsolationLevel`    | The default isolation level for new connections. All out-of-transaction queries are executed with this setting. The isolation levels are available from the `TEDIOUS_ISOLATION_LEVEL` export. (default: `READ_COMMITED`).                                                                                                                                                                                                                                                                           |
| `cryptoCredentialsDetails`    | When `encrypt` is set to true, an object may be supplied that will be used as the secureContext field when creating a [`TLSSocket`](https://nodejs.org/docs/latest/api/tls.html#class-tlstlssocket). The available options are listed under [`tls.createSecureContext`](https://nodejs.org/docs/latest/api/tls.html#tlscreatesecurecontextoptions).                                                                                                                                                 |
| `datefirst`                   | An integer representing the first day of the week. (default: 7)                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `dateFormat`                  | A string representing the date format. (default: `mdy`)                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `debug`                       | See `options.debug` in [the Tedious documentation]                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `enableAnsiNull`              | Controls the way null values should be used during comparison operation. (default: true)                                                                                                                                                                                                                                                                                                                                                                                                            |
| `enableAnsiPadding`           | Controls if padding should be applied for values shorter than the size of defined column. (default: true)                                                                                                                                                                                                                                                                                                                                                                                           |
| `enableAnsiWarnings`          | If true, SQL Server will follow ISO standard behavior during various error conditions. For details, see [documentation](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-ansi-warnings-transact-sql). (default: true)                                                                                                                                                                                                                                                                      |
| `enableArithAbort`            | Ends a query when an overflow or divide-by-zero error occurs during query execution. See [documentation](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-arithabort-transact-sql?view=sql-server-2017) for more details. (default: true)                                                                                                                                                                                                                                                  |
| `enableConcatNullYieldsNull`  | If true, concatenating a null value with a string results in a NULL value. (default: true)                                                                                                                                                                                                                                                                                                                                                                                                          |
| `enableCursorCloseOnCommit`   | If true, cursors will be closed when a transaction is committed or rolled back. (default: null)                                                                                                                                                                                                                                                                                                                                                                                                     |
| `enableImplicitTransactions`  | Sets the connection to either implicit or autocommit transaction mode. (default: false)                                                                                                                                                                                                                                                                                                                                                                                                             |
| `enableNumericRoundabort`     | If false, error is not generated during loss of precession. (default: false)                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `encrypt`                     | A string value set to `'strict'` enables the TDS 8.0 protocol. Otherwise, encrypt can be set to a boolean value which determines whether or not the connection will be encrypted under the TDS 7.x protocol. (default: true)                                                                                                                                                                                                                                                                        |
| `fallbackToDefaultDb`         | By default, if the database requested by options.database cannot be accessed, the connection will fail with an error. However, if this is set to true, then the user's default database will be used instead (Default: false).                                                                                                                                                                                                                                                                      |
| `language`                    | Specifies the language environment for the session. The session language determines the datetime formats and system messages. (default: us_english).                                                                                                                                                                                                                                                                                                                                                |
| `maxRetriesOnTransientErrors` | The maximum number of connection retries for transient errors. (default: 3).                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `multiSubnetFailover`         | Sets the `MultiSubnetFailover = True` parameter, which can help minimize the client recovery latency when failovers occur. (default: false).                                                                                                                                                                                                                                                                                                                                                        |
| `packetSize`                  | The size of TDS packets (subject to negotiation with the server). Should be a power of 2. (default: 4096).                                                                                                                                                                                                                                                                                                                                                                                          |
| `readOnlyIntent`              | A boolean, determining whether the connection will request read only access from a SQL Server Availability Group. For more information, see here. (default: false).                                                                                                                                                                                                                                                                                                                                 |
| `requestTimeout`              | The number of milliseconds before a request is considered failed, or 0 for no timeout (default: 15000).                                                                                                                                                                                                                                                                                                                                                                                             |
| `tdsVersion`                  | The version of TDS to use. If server doesn't support specified version, negotiated version is used instead. The versions are available from the `TDS_VERSION` export. (default: 7_4).                                                                                                                                                                                                                                                                                                               |
| `textsize`                    | Specifies the size of varchar(max), nvarchar(max), varbinary(max), text, ntext, and image data returned by a SELECT statement. (default: 2147483647) (Textsize is set by a numeric value.)                                                                                                                                                                                                                                                                                                          |
| `trustServerCertificate`      | If "true", the SQL Server SSL certificate is automatically trusted when the communication layer is encrypted using SSL. If "false", the SQL Server validates the server SSL certificate. If the server certificate validation fails, the driver raises an error and terminates the connection. Make sure the value passed to serverName exactly matches the Common Name (CN) or DNS name in the Subject Alternate Name in the server certificate for an SSL connection to succeed. (default: true). |

[the Tedious documentation]: https://tediousjs.github.io/tedious/api-connection.html#function_newConnection

### Domain Account

In order to connect with a domain account, use the following format.

```ts
const sequelize = new Sequelize({
  dialect: MsSqlDialect,
  instanceName: 'SQLEXPRESS',
  authentication: {
    type: 'ntlm',
    options: {
      domain: 'yourDomain',
      userName: 'username',
      password: 'password',
    },
  },
});
```
