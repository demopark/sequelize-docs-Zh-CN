# Sequelize Docs 中文版

![](http://docs.sequelizejs.com/manual/asset/logo-small.png)

[![npm version](https://badgen.net/npm/v/@sequelize/core)](https://www.npmjs.com/package/@sequelize/core)
[![npm downloads](https://badgen.net/npm/dm/@sequelize/core)](https://www.npmjs.com/package/@sequelize/core)
[![sponsor](https://img.shields.io/opencollective/all/sequelize?label=sponsors)](https://opencollective.com/sequelize)
[![Last commit](https://badgen.net/github/last-commit/sequelize/sequelize/main)](https://github.com/sequelize/sequelize)
[![Merged PRs](https://badgen.net/github/merged-prs/sequelize/sequelize)](https://github.com/sequelize/sequelize)
[![GitHub stars](https://badgen.net/github/stars/sequelize/sequelize)](https://github.com/sequelize/sequelize)
[![node](https://badgen.net/npm/node/@sequelize/core)](https://www.npmjs.com/package/@sequelize/core)
[![License](https://badgen.net/github/license/sequelize/sequelize)](https://github.com/sequelize/sequelize/blob/main/LICENSE)

> 此项目同步自 [sequelize](https://github.com/sequelize) / [sequelize](https://github.com/sequelize/sequelize) 项目.
> 
> 由于v7现阶段处于alpha阶段, 变更较为频繁, 生产环境建议使用v6

Sequelize 是一个易用且基于 promise 的 [Node.js](https://nodejs.org/en/about/) [ORM 工具](https://en.wikipedia.org/wiki/Object-relational_mapping) 适用于 [Postgres](https://en.wikipedia.org/wiki/PostgreSQL), [MySQL](https://en.wikipedia.org/wiki/MySQL), [MariaDB](https://en.wikipedia.org/wiki/MariaDB), [SQLite](https://en.wikipedia.org/wiki/SQLite), [Microsoft SQL Server](https://en.wikipedia.org/wiki/Microsoft_SQL_Server), [Amazon Redshift](https://docs.aws.amazon.com/redshift/index.html) 和 [Snowflake’s Data Cloud](https://docs.snowflake.com/en/user-guide/intro-key-concepts.html). 它具有强大的事务支持, 关联关系, 预读和延迟加载,读取复制等功能.

Sequelize 遵从 [语义版本控制](http://semver.org) 和 [官方 Node.js LTS 版本](https://nodejs.org/en/about/releases/). Sequelize v7 版本正式支持 Node.js `>=18.0.0`.

你目前正在查看 Sequelize 的**教程和指南**.你可能还对[API 参考](https://sequelize.org/api/v7/) (英文)感兴趣.


![赞赏支持](https://raw.githubusercontent.com/demopark/electron-api-demos-Zh_CN/master/assets/img/td.png)


## 文档版本

- [v7 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/master)(更新中)

- [v6 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v6)(视变更情况更新)

- [v5 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v5)(停止更新)

- [v4 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v4)(停止更新)

## 主要版本变更日志

在此处可以找到主要版本的升级信息：

- [从 v5 升级到 v6](other-topics/upgrade-to-v6.md)
- [从 v6 升级到 v7](other-topics/upgrade-to-v7.md)

## 文档(v7-alpha)

**注意** 由于当前alpha阶段api调整, 文档中的API参考指向尚未确定. 可前往 [V7 API 参考](https://sequelize.org/api/v7/)自行查询.

### 核心概念

- [Getting Started - 入门](core-concepts/getting-started.md)
- [Model Basics - 模型基础](core-concepts/model-basics.md)
- [Model Instances - 模型实例](core-concepts/model-instances.md)
- [Model Querying - Basics - 模型查询(基础)](core-concepts/model-querying-basics.md)
- [Model Querying - Finders - 模型查询(查找器)](core-concepts/model-querying-finders.md)
- [Getters, Setters & Virtuals - 获取器, 设置器 & 虚拟字段](core-concepts/getters-setters-virtuals.md)
- [Validations & Constraints - 验证 & 约束](core-concepts/validations-and-constraints.md)
- [Raw Queries - 原始查询](core-concepts/raw-queries.md)
- [Associations - 关联](core-concepts/assocs.md)
- [Paranoid - 偏执表](core-concepts/paranoid.md)

### 高级关联概念

- [Eager Loading - 预先加载](advanced-association-concepts/eager-loading.md)
- [Creating with Associations - 创建关联](advanced-association-concepts/creating-with-associations.md)
- [Advanced M:N Associations - 高级 M:N 关联](advanced-association-concepts/advanced-many-to-many.md)
- [Association Scopes - 关联作用域](advanced-association-concepts/association-scopes.md)
- [Polymorphic Associations - 多态关联](advanced-association-concepts/polymorphic-associations.md)

### 其它主题

- [Dialect-Specific Things - 方言特定事项](other-topics/dialect-specific-things.md)
- [Transactions - 事务](other-topics/transactions.md)
- [Hooks - 钩子](other-topics/hooks.md)
- [Query Interface - 查询接口](other-topics/query-interface.md)
- [Naming Strategies - 命名策略](other-topics/naming-strategies.md)
- [Scopes - 作用域](other-topics/scopes.md)
- [Sub Queries - 子查询](other-topics/sub-queries.md)
- [Other Data Types - 其他数据类型](other-topics/other-data-types.md)
- [Constraints & Circularities - 约束 & 循环](other-topics/constraints-and-circularities.md)
- [Extending Data Types - 扩展数据类型](other-topics/extending-data-types.md)
- [Indexes - 索引](other-topics/indexes.md)
- [Optimistic Locking - 乐观锁定](other-topics/optimistic-locking.md)
- [Read Replication - 读取复制](other-topics/read-replication.md)
- [Connection Pool - 连接池](other-topics/connection-pool.md)
- [Working with Legacy Tables - 使用遗留表](other-topics/legacy.md)
- [Migrations - 迁移](other-topics/migrations.md)
- [TypeScript](other-topics/typescript.md)
- [Resources - 资源](other-topics/resources.md)

## 简单示例

#### TypeScript

```typescript
import {
  Sequelize,
  Model,
  DataTypes,
  InferAttributes,
  InferCreationAttributes,
} from '@sequelize/core';
import { Attribute } from '@sequelize/core/decorators-legacy';
import { SqliteDialect } from '@sequelize/sqlite3';

class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
  @Attribute(DataTypes.STRING)
  declare username: string | null;

  @Attribute(DataTypes.DATE)
  declare birthday: Date | null;
}

const sequelize = new Sequelize({
  dialect: SqliteDialect,
  models: [User],
});

await sequelize.sync();

const jane = await User.create({
  username: 'janedoe',
  birthday: new Date(1980, 6, 20),
});

console.log(jane.toJSON());
```

#### JavaScript

```typescript
import { Sequelize, Model, DataTypes } from '@sequelize/core';
import { Attribute } from '@sequelize/core/decorators-legacy';
import { SqliteDialect } from '@sequelize/sqlite3';

class User extends Model {
  @Attribute(DataTypes.STRING)
  username;

  @Attribute(DataTypes.DATE)
  birthday;
}

const sequelize = new Sequelize({
  dialect: SqliteDialect,
  models: [User],
});

await sequelize.sync();

const jane = await User.create({
  username: 'janedoe',
  birthday: new Date(1980, 6, 20),
});

console.log(jane.toJSON());
```

请通过 [Getting started - 入门](core-concepts/getting-started.md) 来学习更多相关内容. 如果你想要学习 Sequelize API 请通过 [API 参考](https://sequelize.org/api/v7/) (英文).

## 下载量趋势

[近五年Sequelize下载量趋势](https://npm-compare.com/img/npm-trend/FIVE_YEARS/sequelize.png)

<a href="https://npm-compare.com/sequelize#timeRange=FIVE_YEARS" target="_blank">
  <img src="https://npm-compare.com/img/npm-trend/FIVE_YEARS/sequelize.png" width="100%" alt="NPM Usage Trend of sequelize" />
</a>
