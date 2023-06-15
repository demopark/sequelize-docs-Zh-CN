# Query Interface - 查询接口

Sequelize 实例使用一种称为 **查询接口** 的东西来以与方言无关的方式与数据库进行通信. 你在本手册中学到的大多数方法都是通过查询接口中的几种方法来实现的.

因此,查询接口中的方法是较低级的方法; 仅当找不到其他方法来使用 Sequelize 的高级 API 时,才应使用它们. 当然,它们比直接运行原始查询(即,手工编写SQL)的级别更高.

本指南展示了一些示例,但是要获取其功能的完整列表以及每种方法的详细用法,请查看[查询接口 API](https://sequelize.org/api/v6/class/src/dialects/abstract/query-interface.js~QueryInterface.html).

## 获取查询界面

从现在开始,我们将 `queryInterface` 称为 [查询接口](https://sequelize.org/api/v6/class/src/dialects/abstract/query-interface.js~QueryInterface.html) 类的单例实例,该实例可在你的 Sequelize 实例上使用：

```js
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize(/* ... */);
const queryInterface = sequelize.getQueryInterface();
```

## 创建一个表

```js
queryInterface.createTable('Person', {
  name: DataTypes.STRING,
  isBetaMember: {
    type: DataTypes.BOOLEAN,
    defaultValue: false,
    allowNull: false
  }
});
```

生成 SQL (使用 SQLite):

```SQL
CREATE TABLE IF NOT EXISTS `Person` (
  `name` VARCHAR(255),
  `isBetaMember` TINYINT(1) NOT NULL DEFAULT 0
);
```

**注意:** 考虑定义一个模型,然后调用 `YourModel.sync()`,这是一个较高级别的方法.

## 向表添加列

```js
queryInterface.addColumn('Person', 'petName', { type: DataTypes.STRING });
```

生成 SQL (使用 SQLite):

```sql
ALTER TABLE `Person` ADD `petName` VARCHAR(255);
```

## 更改列的数据类型

```js
queryInterface.changeColumn('Person', 'foo', {
  type: DataTypes.FLOAT,
  defaultValue: 3.14,
  allowNull: false
});
```

生成 SQL (使用 MySQL):

```sql
ALTER TABLE `Person` CHANGE `foo` `foo` FLOAT NOT NULL DEFAULT 3.14;
```

## 删除列

```js
queryInterface.removeColumn('Person', 'petName', { /* 查询参数 */ });
```

生成 SQL (使用 PostgreSQL):

```SQL
ALTER TABLE "public"."Person" DROP COLUMN "petName";
```

## 更改和删除 SQLite 中的列

SQLite 不支持直接更改和删除列. 但是,Sequelize 将通过受[这些说明](https://www.sqlite.org/lang_altertable.html#otheralter)的启发在备份表的帮助下重新创建整个表,以解决此问题.

示例:

```js
// 假设我们在 SQLite 中创建了一个表,如下所示:
queryInterface.createTable('Person', {
  name: DataTypes.STRING,
  isBetaMember: {
    type: DataTypes.BOOLEAN,
    defaultValue: false,
    allowNull: false
  },
  petName: DataTypes.STRING,
  foo: DataTypes.INTEGER
});

// 我们改变一列:
queryInterface.changeColumn('Person', 'foo', {
  type: DataTypes.FLOAT,
  defaultValue: 3.14,
  allowNull: false
});
```

为 SQLite 生成了以下 SQL 调用:

```sql
PRAGMA TABLE_INFO(`Person`);

CREATE TABLE IF NOT EXISTS `Person_backup` (
  `name` VARCHAR(255),
  `isBetaMember` TINYINT(1) NOT NULL DEFAULT 0,
  `foo` FLOAT NOT NULL DEFAULT '3.14',
  `petName` VARCHAR(255)
);

INSERT INTO `Person_backup`
  SELECT
    `name`,
    `isBetaMember`,
    `foo`,
    `petName`
  FROM `Person`;

DROP TABLE `Person`;

CREATE TABLE IF NOT EXISTS `Person` (
  `name` VARCHAR(255),
  `isBetaMember` TINYINT(1) NOT NULL DEFAULT 0,
  `foo` FLOAT NOT NULL DEFAULT '3.14',
  `petName` VARCHAR(255)
);

INSERT INTO `Person`
  SELECT
    `name`,
    `isBetaMember`,
    `foo`,
    `petName`
  FROM `Person_backup`;

DROP TABLE `Person_backup`;
```

## 其它

如本指南开头所述,Sequelize 中的查询接口还有很多！ 查看 [查询接口 API](https://sequelize.org/api/v6/class/src/dialects/abstract/query-interface.js~QueryInterface.html),以获取可以完成的操作的完整列表.