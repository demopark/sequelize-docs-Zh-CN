# Sequelize 用于 PostgreSQL



请参阅 [Releases](/releases#postgresql-support-table) 以了解支持哪些版本的 PostgreSQL。


要在 PostgreSQL 上使用 Sequelize，你需要安装 `@sequelize/postgres` 方言包：

```bash npm2yarn
npm i @sequelize/postgres
```


然后在 Sequelize 构造函数中使用 `PostgresDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { PostgresDialect } from '@sequelize/postgres';

const sequelize = new Sequelize({
  dialect: PostgresDialect,
  database: 'mydb',
  user: 'myuser',
  password: 'mypass',
  host: 'localhost',
  port: 5432,
  ssl: true,
  clientMinMessages: 'notice',
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


PostgreSQL 方言支持以下选项：

| 选项                                  | 说明                                                                                                                                                                                                                                                                       |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `database`                            | 要连接的数据库名称。                                                                                                                                                                                                                                                       |
| `user`                                | 用于认证的用户名。                                                                                                                                                                                                                                                         |
| `password`                            | 用户密码。                                                                                                                                                                                                                                                                |
| `host`                                | 要连接的主机（可以是 IP、域名或 Unix 套接字路径）。                                                                                                                                                                                |
| `port`                                | 要连接的端口，默认是 `5432`。                                                                                                                                                                                                      |
| `ssl`                                 | 连接服务器时使用的 SSL 配置。会直接传递给 [`TLSSocket`](https://nodejs.org/docs/latest/api/tls.html#class-tlstlssocket)，支持所有 [`tls.connect`](https://nodejs.org/docs/latest/api/tls.html#tlsconnectoptions-callback) 选项。 |
| `query_timeout`                       | 查询调用超时时间（毫秒），默认无限制。                                                                                                                                                                                            |
| `connectionTimeoutMillis`             | 连接超时时间（毫秒），默认无限制。                                                                                                                                                                                                |
| `application_name`                    | 配置连接的 [PostgreSQL `application_name` 选项](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-APPLICATION-NAME)。                                                 |
| `statement_timeout`                   | 配置连接的 [PostgreSQL `statement_timeout` 选项](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT)。                                               |
| `idle_in_transaction_session_timeout` | 配置连接的 [PostgreSQL `idle_in_transaction_session_timeout` 选项](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-IDLE-IN-TRANSACTION-SESSION-TIMEOUT)。           |
| `client_encoding`                     | 配置连接的 [PostgreSQL `client_encoding` 选项](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-ENCODING)，默认 `utf8`。                                      |
| `lock_timeout`                        | 配置连接的 [PostgreSQL `lock_timeout` 选项](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LOCK-TIMEOUT)。                                                         |
| `options`                             | Configures [the `options` libpq option](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNECT-OPTIONS) for the connection.                                                                                                                              |
| `keepAlive`                           | Configures [the libpq `keepalives` option](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-KEEPALIVES). Must be a boolean.                                                                                                                                |
| `keepAliveInitialDelayMillis`         | Configures [the libpq `keepalives_idle` option](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-KEEPALIVES-IDLE), but in milliseconds (will be rounded down to the nearest second).                                                                       |

:::info

Sequelize uses the `pg` package to connect to PostgreSQL.
Most of the above options are provided as-is to the `pg` package,
and you can find more information about them in the [pg documentation](https://node-postgres.com/apis/client#new-client).

:::

### Connection via Unix Socket

To connect to PostgreSQL using a Unix socket, you can use the `host` option with the absolute path to the socket file:

```ts
const sequelize = new Sequelize({
  dialect: PostgresDialect,
  host: '/var/run/postgresql',
});
```

## Other PostgreSQL Options

The following options are also available for PostgreSQL:

| Option                      | Description                                                                                                                                                                                                                                                                                                                                                                     |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `clientMinMessages`         | Configures [the `client_min_messages` PostgreSQL option](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-MIN-MESSAGES) for all connections. Defaults to "warning".                                                                                                                                                                                |
| `standardConformingStrings` | Configures [the `standard_conforming_strings` PostgreSQL option](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-STANDARD-CONFORMING-STRINGS) for all connections. If your PostgreSQL server is configured with `standard_conforming_strings = off`, it is extremely important to set this option to `false` to avoid SQL injection vulnerabilities. |
| `native`                    | If true, Sequelize will use the `pg-native` package instead of the `pg` package. `pg-native` must be installed separately.                                                                                                                                                                                                                                                      |

## Amazon Redshift

:::caution

While Redshift is based on PostgreSQL, it does not support the same set of features as PostgreSQL.

As we do not have access to a Redshift instance, we cannot guarantee that Sequelize will work correctly with Redshift,
and we rely on the help of the community to keep this documentation up to date.

:::

Redshift doesn't support `client_min_messages`, you must set it to `'ignore'`:

```ts
new Sequelize({
  dialect: PostgresDialect,
  // Your pg options here
  clientMinMessages: 'ignore',
});
```
