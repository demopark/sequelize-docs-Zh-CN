# Sequelize 用于 DB2 for Linux、Unix 和 Windows



请参阅 [Releases](/releases#db2-for-luw-support-table) 以了解支持哪些版本的 DB2 for LUW。


要在 DB2 for LUW 上使用 Sequelize，你需要安装 `@sequelize/db2` 方言包：

```bash npm2yarn
npm i @sequelize/db2
```


然后在 Sequelize 构造函数中使用 `Db2Dialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { Db2Dialect } from '@sequelize/db2';

const sequelize = new Sequelize({
  dialect: Db2Dialect,
  database: 'mydb',
  user: 'myuser',
  password: 'mypass',
  hostname: 'localhost',
  port: 50000,
  ssl: true,
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


DB2 for LUW 方言支持以下选项：

| 选项                    | 说明                                            |
| ---------------------- | ----------------------------------------------- |
| `database`             | ODBC "DATABASE" 参数                            |
| `username`             | ODBC "UID" 参数                                 |
| `password`             | ODBC "PWD" 参数                                 |
| `hostname`             | ODBC "HOSTNAME" 参数                            |
| `port`                 | ODBC "PORT" 参数                                |
| `ssl`                  | 当为 true 时，将 ODBC "Security" 参数设置为 SSL  |
| `sslServerCertificate` | ODBC "SSLServerCertificate" 参数                |
| `odbcOptions`          | 其他 ODBC 参数                                   |
