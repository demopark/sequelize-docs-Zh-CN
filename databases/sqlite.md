# Sequelize 用于 SQLite


要在 SQLite 上使用 Sequelize，你需要安装 `@sequelize/sqlite` 方言包：

```bash npm2yarn
npm i @sequelize/sqlite3
```


然后在 Sequelize 构造函数中使用 `SqliteDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { SqliteDialect } from '@sequelize/sqlite3';

const sequelize = new Sequelize({
  dialect: SqliteDialect,
  storage: 'sequelize.sqlite',
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


SQLite 方言支持以下选项：

| 选项       | 说明                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `storage`  | SQLite 数据库文件的路径。                                                                                                                                                                                                                                                                                                                                      |
| `mode`     | 用于打开数据库连接的整型位标志。<br /><br />该模式可以是 `OPEN_READONLY`、`OPEN_READWRITE`、`OPEN_CREATE`、`OPEN_FULLMUTEX`、`OPEN_URI`、`OPEN_SHAREDCACHE`、`OPEN_PRIVATECACHE` 的按位或（使用 `|`），这些常量均由 `@sequelize/sqlite` 提供。<br/><br />具体含义请参考 [SQLite 官方文档](https://www.sqlite.org/c3ref/open.html)。 |
| `password` | 连接时使用的 "PRAGMA KEY" 密码（如使用 `sqlcipher` 等插件）。                                                                                                                                                                                                                                                        |


### 临时存储

SQLite 支持两种类型的临时存储：

- 将 `storage` 设为空字符串，使用基于磁盘的临时存储。
- 将 `storage` 设为 `':memory:'`，使用基于内存的临时存储。

无论哪种方式，数据库在连接关闭时都会被销毁。因此，使用临时存储时需要配置 [连接池](../other-topics/connection-pool.md) 保持单一连接，配置如下：

```ts
const sequelize = new Sequelize({
  dialect: SqliteDialect,
  storage: ':memory:', // or ''
  pool: { max: 1, idle: Infinity, maxUses: Infinity },
});
```


## 其他 SQLite 选项

SQLite 还支持以下选项：

| Option        | Description                                            |
| ------------- | ------------------------------------------------------ |
| `foreignKeys` | If set to false, SQLite will not enforce foreign keys. |
