# Sequelize Docs 中文版

![](http://docs.sequelizejs.com/manual/asset/logo-small.png)

[![Travis build](https://img.shields.io/travis/sequelize/sequelize/master.svg?style=flat-square)](https://travis-ci.org/sequelize/sequelize)
[![npm](https://img.shields.io/npm/dm/sequelize.svg?style=flat-square)](https://npmjs.org/package/sequelize)
[![npm](https://img.shields.io/npm/v/sequelize.svg?style=flat-square)](https://github.com/sequelize/sequelize/releases)

> 此项目同步自 [sequelize](https://github.com/sequelize) / [sequelize](https://github.com/sequelize/sequelize) 项目中的  docs. 除特殊情况, 将保持每月一次的同步频率.
> 
> 更新日志请参阅: [CHANGELOG](CHANGELOG.md)

Sequelize 是一个基于 promise 的 Node.js ORM, 目前支持 Postgres, MySQL, SQLite 和 Microsoft SQL Server. 它具有强大的事务支持, 关联关系, 读取和复制等功能.

## 版本

### [v6 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/master)

### [v5 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v5)

### [v4 中文文档](https://github.com/demopark/sequelize-docs-Zh-CN/tree/v4)(停止更新)

## 文档

- [Getting started - 入门](getting-started.md)
- [Model definition - 模型定义](models-definition.md)
- [Model usage - 模型使用](models-usage.md)
- [Querying - 查询](querying.md)
- [Instances - 实例](instances.md)
- [Associations - 关联](associations.md)
- [Transactions - 事务](transactions.md)
- [Scopes - 作用域](scopes.md)
- [Hooks - 钩子](hooks.md)
- [Raw queries - 原始查询](raw-queries.md)
- [Migrations - 迁移](migrations.md)
- [Upgrade to V4 - 升级到 V4](upgrade-to-v4.md)
- [Working with legacy tables - 使用遗留表](legacy.md)

## 使用示例

[Basic usage - 基本用法](usage.md)

```js
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },

  // 仅限 SQLite
  storage: 'path/to/database.sqlite',

  // 请参考 Querying - 查询 操作符 章节
  operatorsAliases: false
});

const User = sequelize.define('user', {
  username: Sequelize.STRING,
  birthday: Sequelize.DATE
});

sequelize.sync()
  .then(() => User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  }))
  .then(jane => {
    console.log(jane.toJSON());
  });
```

请通过 [Getting started - 入门](getting-started.md) 来学习更多相关内容。 如果你想要学习 Sequelize API 请通过 [API Reference](http://docs.sequelizejs.com/identifiers) (英文)。

# 赞赏支持
![赞赏支持](https://raw.githubusercontent.com/demopark/electron-api-demos-Zh_CN/master/assets/img/td.png)
