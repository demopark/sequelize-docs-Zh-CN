# Constraints & Circularities - 约束 & 循环

在表之间添加约束意味着使用 `sequelize.sync` 时必须在数据库中以一定顺序创建表. 如果 `Task` 具有对 `User` 的引用,则必须先创建 `User` 表,然后才能创建 `Task` 表. 有时这可能会导致循环引用,而 Sequelize 无法找到同步的顺序. 想象一下文档和版本的情况. 一个文档可以有多个版本,为方便起见,文档引用了其当前版本.

```js
const { Sequelize, Model, DataTypes } = require('@sequelize/core');

class Document extends Model {}
Document.init({
    author: DataTypes.STRING
}, { sequelize, modelName: 'document' });

class Version extends Model {}
Version.init({
  timestamp: DataTypes.DATE
}, { sequelize, modelName: 'version' });

Document.hasMany(Version); // 这会将 documentId 属性添加到 version 中
Document.belongsTo(Version, {
  as: 'Current',
  foreignKey: 'currentVersionId'
}); // 这会将 currentVersionId 属性添加到 document 中
```

但是,不幸的是,上面的代码将导致以下错误:

```text
Cyclic dependency found. documents is dependent of itself. Dependency chain: documents -> versions => documents
```

为了减少这种情况,我们可以将 `constraints: false` 传递给关联之一:

```js
Document.hasMany(Version);
Document.belongsTo(Version, {
  as: 'Current',
  foreignKey: 'currentVersionId',
  constraints: false
});
```

这将使我们能够正确同步表:

```sql
CREATE TABLE IF NOT EXISTS "documents" (
  "id" SERIAL,
  "author" VARCHAR(255),
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "currentVersionId" INTEGER,
  PRIMARY KEY ("id")
);

CREATE TABLE IF NOT EXISTS "versions" (
  "id" SERIAL,
  "timestamp" TIMESTAMP WITH TIME ZONE,
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "documentId" INTEGER REFERENCES "documents" ("id") ON DELETE
  SET
    NULL ON UPDATE CASCADE,
    PRIMARY KEY ("id")
);
```

## 不受限制地强制执行外键引用

有时,你可能希望引用另一个表,而不添加任何约束或关联. 在这种情况下,你可以将引用属性手动添加到架构定义中,并标记它们之间的关系.

```js
class Trainer extends Model {}
Trainer.init({
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING
}, { sequelize, modelName: 'trainer' });

// 在我们调用 Trainer.hasMany(series) 之后,
// Series 将会有一个 trainerId = Trainer.id 外参考键
class Series extends Model {}
Series.init({
  title: DataTypes.STRING,
  subTitle: DataTypes.STRING,
  description: DataTypes.TEXT,
  // 设置与 `Trainer` 的外键关系(hasMany)
  trainerId: {
    type: DataTypes.INTEGER,
    references: {
      model: Trainer,
      key: 'id'
    }
  }
}, { sequelize, modelName: 'series' });

// 在我们调用 Series.hasOne(Video) 之后,
// Video 将具有 seriesId = Series.id 外参考键
class Video extends Model {}
Video.init({
  title: DataTypes.STRING,
  sequence: DataTypes.INTEGER,
  description: DataTypes.TEXT,
  // 设置与 `Series` 的关系(hasOne)
  seriesId: {
    type: DataTypes.INTEGER,
    references: {
      model: Series, // 可以是代表表名的字符串,也可以是 Sequelize 模型
      key: 'id'
    }
  }
}, { sequelize, modelName: 'video' });

Series.hasOne(Video);
Trainer.hasMany(Series);
```