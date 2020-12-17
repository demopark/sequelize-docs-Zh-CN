# Extending Data Types - 扩展数据类型

你尝试实现的类型很可能已经包含在[数据类型](../core-concepts/model-basics.md)中. 如果不包括新的数据类型,本手册将说明如何自己编写它.

Sequelize 不会在数据库中创建新的数据类型. 本教程说明了如何使 Sequelize 识别新数据类型,并假定这些新数据类型已在数据库中创建.

要扩展 Sequelize 数据类型,请在创建 Sequelize 实例之前进行.

## 示例

在此示例中,我们将创建一个名为 `SOMETYPE` 的类型,该类型将复制内置数据类型 `DataTypes.INTEGER(11).ZEROFILL.UNSIGNED`.

```js
const { Sequelize, DataTypes, Utils } = require('Sequelize');
createTheNewDataType();
const sequelize = new Sequelize('sqlite::memory:');

function createTheNewDataType() {

  class SOMETYPE extends DataTypes.ABSTRACT {
    // 强制性的: 在数据库中完整定义新类型
    toSql() {
      return 'INTEGER(11) UNSIGNED ZEROFILL'
    }

    // 可选的: 验证器功能
    validate(value, options) {
      return (typeof value === 'number') && (!Number.isNaN(value));
    }

    // 可选的: sanitizer
    _sanitize(value) {
      // 强制所有数字为正
      return value < 0 ? 0 : Math.round(value);
    }

    // 可选的: 发送到数据库之前的值字符串化
    _stringify(value) {
      return value.toString();
    }

    // 可选的: 解析器,用于从数据库接收的值
    static parse(value) {
      return Number.parseInt(value);
    }
  }

  // 强制性的: 设置类型键
  SOMETYPE.prototype.key = SOMETYPE.key = 'SOMETYPE';

  // 强制性的: 将新类型添加到数据类型. 将其包装在 `Utils.classToInvokable` 上,
  // 以能够直接使用此数据类型,而不必调用 `new`.
  DataTypes.SOMETYPE = Utils.classToInvokable(SOMETYPE);

  // 可选的: 禁用字符串化后的转义. 这样做需要你自担风险,因为这为 SQL 注入提供了机会.
  // DataTypes.SOMETYPE.escape = false;

}
```

创建此新数据类型后,你需要在每个数据库方言中映射此数据类型并进行一些调整.

## PostgreSQL

假设新数据类型的名称在 postgres 数据库中为 `pg_new_type`. 该名称必须映射到 `DataTypes.SOMETYPE`. 此外,还需要创建特定于 Postgres 的子数据类型.

```js
function createTheNewDataType() {
  // [...]

  const PgTypes = DataTypes.postgres;

  // 强制性的: 映射 postgres 数据类型名称
  DataTypes.SOMETYPE.types.postgres = ['pg_new_type'];

  // 强制性的: 使用自己的解析方法创建特定于 postgres 的子数据类型.
  // 解析器将动态映射到 pg_new_type 的 OID.
  PgTypes.SOMETYPE = function SOMETYPE() {
    if (!(this instanceof PgTypes.SOMETYPE)) {
      return new PgTypes.SOMETYPE();
    }
    DataTypes.SOMETYPE.apply(this, arguments);
  }
  const util = require('util'); // Node 包内置
  util.inherits(PgTypes.SOMETYPE, DataTypes.SOMETYPE);

  // 强制性的: 创建,覆盖或重新分配特定于 Postgres 的解析器
  // PgTypes.SOMETYPE.parse = value => value;
  PgTypes.SOMETYPE.parse = DataTypes.SOMETYPE.parse || x => x;

  // 可选的: 添加或覆盖特定于Postgres数据类型的方法,
  // 例如 toSql, escape, validate, _stringify, _sanitize...

}
```

### 范围

在[postgres中定义](https://www.postgresql.org/docs/current/static/rangetypes.html#RANGETYPES-DEFINING)了新的范围类型后,将其添加到 Sequelize 变得很简单.

在此示例中,postgres 范围类型的名称为 `SOMETYPE_range`,基础 postgres 数据类型的名称为 `pg_new_type`. `subtypes` 和 `castTypes` 的键是 Sequelize 数据类型 `DataTypes.SOMETYPE.key` 的键(小写).

```js
function createTheNewDataType() {
  // [...]

  // 添加 postgresql 范围,SOMETYPE 来自 DataType.SOMETYPE.key(小写)
  DataTypes.RANGE.types.postgres.subtypes.SOMETYPE = 'SOMETYPE_range';
  DataTypes.RANGE.types.postgres.castTypes.SOMETYPE = 'pg_new_type';
}
```

新范围可在模型定义中用作 `DataTypes.RANGE(DataTypes.SOMETYPE)` 或 `DataTypes.RANGE(DataTypes.SOMETYPE)`.
