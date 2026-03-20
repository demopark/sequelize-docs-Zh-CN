sidebar_position: 1000

# 术语表

## `Sequelize` 与 `sequelize`

在我们的文档中，`Sequelize` 指的是该库本身，而小写的 `sequelize` 指的是 Sequelize 的一个实例。
这是推荐的命名规范，整个文档都会遵循这一约定。

## 表、模型与实体

在 Sequelize 中，模型（Models）是代表[数据库表](<https://en.wikipedia.org/wiki/Table_(database)>)的 JavaScript 类。

有些 ORM 将其称为实体（Entities）。在 Sequelize 中，按照惯例称为模型（Models）。

更多关于模型的信息请参见[定义模型](./models/defining-models.md)

## 属性与列

属性（Attribute）是[表列](<https://en.wikipedia.org/wiki/Column_(database)>)在 JavaScript 中的表示。

在本文件中，如果使用 _column_，指的是 SQL 列；如果使用 _attribute_，指的是列在 JavaScript 中的表示。

更多关于属性的信息请参见[定义模型](./models/defining-models.md)

## 关联与关系

关联（Association）是[表关系](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model)在 JavaScript 中的表示。

更多信息请参见[关联](./associations/basics.md)
