# Sequelize 用于 Snowflake



我们对 Snowflake 的实现并未在真实数据库上进行集成测试。
因此，我们无法保证其能如预期工作，也无法保证其稳定性。

我们依赖社区的帮助来改进此方言。



请参阅 [Releases](/releases#snowflake-support-table) 以了解支持哪些版本的 Snowflake。


要在 Snowflake 上使用 Sequelize，你需要安装 `@sequelize/snowflake` 方言包：

```bash npm2yarn
npm i @sequelize/snowflake
```


然后在 Sequelize 构造函数中使用 `SnowflakeDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { SnowflakeDialect } from '@sequelize/snowflake';

const sequelize = new Sequelize({
  dialect: SnowflakeDialect,
  accessUrl: 'https://myaccount.us-east-1.snowflakecomputing.com',
  role: 'myRole',
  warehouse: 'myWarehouse',
  username: 'myUserName',
  password: 'myPassword',
  database: 'myDatabaseName',
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


以下选项会原样传递给 Sequelize 用于连接 Snowflake 的 `snowflake-sdk` 包。
更多关于每个选项的作用，请参考 [Snowflake 官方文档](https://docs.snowflake.com/en/developer-guide/node-js/nodejs-driver-options)。

为方便起见，下面是仅包含 Sequelize 支持选项的文档摘录：

| 选项                                       | 说明                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `accessUrl`                                | 指定用于连接 Snowflake 的完整端点。                                                                                                                                                                                                                                                                                                                                                           |
| `account`                                  | Snowflake 账户标识符                                                                                                                                                                                                                                                                                                                                                                         |
| `application`                              | Specifies the name of the client application connecting to Snowflake.                                                                                                                                                                                                                                                                                                                                                                 |
| `authenticator`                            | Specifies the authenticator to use for verifying user login credentials. See the [Snowflake documentation](https://docs.snowflake.com/en/developer-guide/node-js/nodejs-driver-options#authentication-options) for details                                                                                                                                                                                                            |
| `clientSessionKeepAlive`                   | By default, client connections typically time out approximately 3-4 hours after the most recent query was executed.<br/>If the `clientSessionKeepAlive` option is set to `true`, the client’s connection to the server will be kept alive indefinitely, even if no queries are executed.<br/>Note that [the Sequelize Pool](../other-topics/connection-pool.md) also disconnects connections if they are idle after a specified time. |
| `clientSessionKeepAliveHeartbeatFrequency` | Sets the frequency (interval in seconds) between heartbeat messages.                                                                                                                                                                                                                                                                                                                                                                  |
| `database`                                 | The default database to use for the session after connecting.                                                                                                                                                                                                                                                                                                                                                                         |
| `password`                                 | Password for the user for when `authenticator` is set to `SNOWFLAKE`.                                                                                                                                                                                                                                                                                                                                                                 |
| `privateKey`                               | Specifies the private key (in PEM format) for key pair authentication.                                                                                                                                                                                                                                                                                                                                                                |
| `privateKeyPass`                           | Specifies the passcode to decrypt the private key file, if the file is encrypted.                                                                                                                                                                                                                                                                                                                                                     |
| `privateKeyPath`                           | Specifies the local path to the private key file (e.g. `rsa_key.p8`).                                                                                                                                                                                                                                                                                                                                                                 |
| `role`                                     | The default security role to use for the session after connecting.                                                                                                                                                                                                                                                                                                                                                                    |
| `timeout`                                  | Number of milliseconds to keep the connection alive with no response. Default: 60000 (1 minute).                                                                                                                                                                                                                                                                                                                                      |
| `token`                                    | Specifies the OAuth token to use for authentication for when `authenticator` is set to `OAUTH`.                                                                                                                                                                                                                                                                                                                                       |
| `username`                                 | The login name for your Snowflake user or your Identity Provider.                                                                                                                                                                                                                                                                                                                                                                     |
| `warehouse`                                | The default virtual warehouse to use for the session after connecting. Used for performing queries, loading data, etc.                                                                                                                                                                                                                                                                                                                |

## Other Snowflake Options

The following options are also available for Snowflake:

| Option         | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `showWarnings` | If `true`, warnings produced during the execution of a query will be sent to the `logging` callback. Default is `false`. |
