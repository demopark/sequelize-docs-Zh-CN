# 快速开始

在本教程中，你将学习如何对 Sequelize 进行一个简单的基础配置。

## 安装

Sequelize 可以通过 [npm](https://www.npmjs.com/package/@sequelize/core)（或 [yarn](https://yarnpkg.com/package/@sequelize/core)）安装。

以下命令将安装 Sequelize v7。
如果你在寻找 Sequelize v6（包名为 `sequelize`，而不是 `@sequelize/core`），请
[访问 v6 文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v6)

```bash
# 这将安装 Sequelize 7（Sequelize 最新的 alpha 版本）
npm i @sequelize/core@alpha
```

## 连接数据库

要连接数据库，你必须创建一个 Sequelize 实例，并向其传入数据库配置信息，例如方言（dialect）、主机（host）以及用户名和密码。

不同的方言支持的选项集合不同。请通过下面的链接查看如何连接到你的数据库：

- [PostgreSQL](./databases/postgres.md)
- [MySQL](./databases/mysql.md)
- [MariaDB](./databases/mariadb.md)
- [SQLite](./databases/sqlite.md)
- [Microsoft SQL Server](./databases/mssql.md)
- [DB2 for LUW](./databases/db2.md)
- [DB2 for IBM i](./databases/ibmi.md)
- [Snowflake](./databases/snowflake.md)

如果你的数据库未在上方列表中，Sequelize 默认并不直接支持它。
不过 Sequelize 具有可扩展性：你可以创建自己的方言，或寻找社区维护的方言实现。
更多信息请参阅 [支持其他数据库](./databases/new.md)。

下面是一个连接 SQLite 数据库的简短示例：

```javascript
import { Sequelize } from '@sequelize/core';
import { SqliteDialect } from '@sequelize/sqlite3';

const sequelize = new Sequelize({
  dialect: SqliteDialect,
});
```

`Sequelize` 构造函数接受许多选项。
它们都记录在 [API 参考](https://sequelize.org/api/v7/classes/_sequelize_core.index.Sequelize.html#constructor) 中。

### 测试连接

你可以使用 `.authenticate()` 函数来测试连接是否正常。
注意这完全是可选的，但我们推荐这么做，因为 Sequelize 会在首次连接时获取你的数据库版本。
随后它会使用该版本来判断哪些 SQL 功能可用。

```js
try {
  await sequelize.authenticate();
  console.log('Connection has been established successfully.');
} catch (error) {
  console.error('Unable to connect to the database:', error);
}
```

### 关闭连接

Sequelize 使用连接池来管理数据库连接。这意味着即使你已经不再使用它们，某些连接仍可能保持打开状态。
如果你希望优雅地关闭应用程序，可以使用 [`sequelize.close()`](https://sequelize.org/api/v7/classes/_sequelize_core.index.Sequelize.html#close) 来关闭所有活动连接。

一旦调用了 `sequelize.close()`，就无法再打开新的连接。若要再次访问数据库，你需要创建一个新的 Sequelize 实例。

## 术语约定

请注意，在上面的示例中，`Sequelize` 指的是这个库本身，而小写的 `sequelize` 指的是 Sequelize 的一个实例。
这是我们推荐的命名约定，本文档将始终遵循这一约定。

## TypeScript

Sequelize 提供内置的 TypeScript 支持。

请前往 [版本策略页面](/releases) 了解支持哪些 TypeScript 版本，
并确保你的项目已安装与你的 Node.js 版本相对应的 [`@types/node`](https://www.npmjs.com/package/@types/node) 包。

Sequelize 使用了 `package.json` 中的 [`exports`](https://nodejs.org/api/packages.html#exports) 字段。
你可能需要更新 `tsconfig.json` 配置，以确保 TypeScript 的模块解析支持它。

在撰写本文时（TS 5.4），可以通过两种方式实现：

- Set [`moduleResolution`](https://www.typescriptlang.org/tsconfig#moduleResolution) to `node16`, `nodenext`, or `bundler`,
- 或将 [`resolvePackageJsonExports`](https://www.typescriptlang.org/tsconfig#resolvePackageJsonExports) 设置为 `true`。

## CommonJS 还是 ESM？

本文档大量使用 ECMAScript Modules（ESM），但 Sequelize 也完全支持 CommonJS。
要在 CommonJS 项目中使用 Sequelize，只需用 `require` 替代 `import`：

```js
// 在 ESM 中如何导入 Sequelize
import { Sequelize, Op, Model, DataTypes } from '@sequelize/core';

// 在 CommonJS 中如何导入 Sequelize
const { Sequelize, Op, Model, DataTypes } = require('@sequelize/core');
```

Sequelize 提供的大多数方法都是异步的，因此会返回 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。
我们强烈建议使用 **ECMAScript Modules**，因为它能让你使用 [Top-Level Await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)，而我们在示例中也大量使用了它。
请参阅 [Node.js 文档](https://nodejs.org/api/esm.html) 了解如何在 Node.js 中使用 ESM。

## 日志

为了便于调试，你可以在 Sequelize 中启用日志。做法是将 `logging` 选项设置为一个函数，每当 Sequelize 需要输出日志时就会执行该函数。

示例：

```js
const sequelize = new Sequelize({
  dialect: SqliteDialect,

  // 禁用日志（默认）
  logging: false,

  // 将日志输出到控制台
  logging: console.log,

  // 你也可以使用任意函数，例如把日志发送到日志工具
  logging: (...msg) => console.log(msg),
});
```

如果你只需要记录特定查询，可以在所有会执行查询的模型方法中使用 `logging` 选项，
以及在所有 `queryInterface` 方法中使用它。

## 下一步

现在你已经有了一个 Sequelize 实例，接下来可以从 [定义你的第一个模型](./models/defining-models.md) 开始。
