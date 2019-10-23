# Datatypes - 数据类型

以下是 sequelize 支持的一些数据类型. 有关完整和更新的列表请参阅[数据类型](/master/variable/index.html#static-variable-DataTypes).

```js
Sequelize.STRING                      // VARCHAR(255)
Sequelize.STRING(1234)                // VARCHAR(1234)
Sequelize.STRING.BINARY               // VARCHAR BINARY
Sequelize.TEXT                        // TEXT
Sequelize.TEXT('tiny')                // TINYTEXT
Sequelize.CITEXT                      // CITEXT      仅 PostgreSQL 和 SQLite.

Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.BIGINT(11)                  // BIGINT(11)

Sequelize.FLOAT                       // FLOAT
Sequelize.FLOAT(11)                   // FLOAT(11)
Sequelize.FLOAT(11, 10)               // FLOAT(11,10)

Sequelize.REAL                        // REAL        仅 PostgreSQL.
Sequelize.REAL(11)                    // REAL(11)    仅 PostgreSQL.
Sequelize.REAL(11, 12)                // REAL(11,12) 仅 PostgreSQL.

Sequelize.DOUBLE                      // DOUBLE
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 10)              // DOUBLE(11,10)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)

Sequelize.DATE                        // mysql / sqlite 为 DATETIME, postgres 为带时区的 TIMESTAMP
Sequelize.DATE(6)                     // DATETIME(6) 适用 mysql 5.6.4+. 小数秒支持最多6位精度
Sequelize.DATEONLY                    // DATE 不带时间.
Sequelize.BOOLEAN                     // TINYINT(1)

Sequelize.ENUM('value 1', 'value 2')  // 一个允许值为'value 1'和'value 2'的ENUM
Sequelize.ARRAY(Sequelize.TEXT)       // 定义一个数组. 仅 PostgreSQL.
Sequelize.ARRAY(Sequelize.ENUM)       // 定义一个ENUM数组. 仅 PostgreSQL.

Sequelize.JSON                        // JSON 列. 仅 PostgreSQL, SQLite 和 MySQL.
Sequelize.JSONB                       // JSONB 列. 仅 PostgreSQL.

Sequelize.BLOB                        // BLOB (PostgreSQL 为 bytea)
Sequelize.BLOB('tiny')                // TINYBLOB (PostgreSQL 为 bytea. 其余参数是 medium 和 long)

Sequelize.UUID                        // PostgreSQL 和 SQLite 的 UUID 数据类型,MySQL 的 CHAR(36) BINARY(使用defaultValue:Sequelize.UUIDV1 或 Sequelize.UUIDV4 来让 sequelize 自动生成 id).

Sequelize.CIDR                        // PostgreSQL 的 CIDR 数据类型
Sequelize.INET                        // PostgreSQL 的 INET 数据类型
Sequelize.MACADDR                     // PostgreSQL 的 MACADDR 数据类型

Sequelize.RANGE(Sequelize.INTEGER)    // 定义 int4range 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.BIGINT)     // 定义 int8range 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.DATE)       // 定义 tstzrange 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.DATEONLY)   // 定义 daterange 范围. 仅 PostgreSQL.
Sequelize.RANGE(Sequelize.DECIMAL)    // 定义 numrange 范围. 仅 PostgreSQL.

Sequelize.ARRAY(Sequelize.RANGE(Sequelize.DATE)) // 定义 tstzrange 范围的数组. 仅 PostgreSQL.

Sequelize.GEOMETRY                    // Spatial 列. 仅 PostgreSQL (带有 PostGIS) 或 MySQL.
Sequelize.GEOMETRY('POINT')           // 带有 geometry 类型的 spatial 列. 仅 PostgreSQL (带有 PostGIS) 或 MySQL.
Sequelize.GEOMETRY('POINT', 4326)     // 具有 geometry 类型和 SRID 的 spatial 列. 仅 PostgreSQL (带有 PostGIS) 或 MySQL.
```

BLOB 数据类型允许你以字符串和 buffer 的形式插入数据. 当你在具有 BLOB 列的模型上执行 find 或 findAll 时,该数据将始终作为 buffer 返回.

如果你正在使用 PostgreSQL 不带时区的 TIMESTAMP 并且你需要将其解析为不同的时区,请使用 pg 库自己的解析器:

```js
require('pg').types.setTypeParser(1114, stringValue => {
  return new Date(stringValue + '+0000');
  // 例如,UTC偏移. 使用你想要的任何偏移量.
});
```

除了上面提到的类型之外,integer,bigint,float 和 double 还支持 unsigned 和 zerofill 属性,这些属性可以按任何顺序组合:
请注意,这不适用于 PostgreSQL！

```js
Sequelize.INTEGER.UNSIGNED              // INTEGER UNSIGNED
Sequelize.INTEGER(11).UNSIGNED          // INTEGER(11) UNSIGNED
Sequelize.INTEGER(11).ZEROFILL          // INTEGER(11) ZEROFILL
Sequelize.INTEGER(11).ZEROFILL.UNSIGNED // INTEGER(11) UNSIGNED ZEROFILL
Sequelize.INTEGER(11).UNSIGNED.ZEROFILL // INTEGER(11) UNSIGNED ZEROFILL
```
_以上示例仅显示整数,但使用 bigint 和 float 可以完成相同的操作_

对象表示法中的用法:

```js
// 对于枚举:
class MyModel extends Model {}
MyModel.init({
  states: {
    type: Sequelize.ENUM,
    values: ['active', 'pending', 'deleted']
  }
}, { sequelize })
```

### 数组(ENUM)

只支持PostgreSQL.

Array(枚举)类型需要特殊处理. 每当Sequelize与数据库通信时,它必须使用ENUM名称对数组值进行类型转换.

所以这个枚举名必须遵循这个模式`enum_<table_name>_<col_name>`. 如果你使用`sync`,则会自动生成正确的名称.

### Range 类型

由于 range 类型具有针对其绑定 inclusion/exclusion 的额外信息,因此不能非常简单地使用元组在javascript中表示它们.

提供 range 作为值时,你可以从以下API中进行选择:

```js
// 默认为 '["2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'
// 包含下限,不包含上限
Timeline.create({ range: [new Date(Date.UTC(2016, 0, 1)), new Date(Date.UTC(2016, 1, 1))] });

// 控制包含
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  { value: new Date(Date.UTC(2016, 1, 1)), inclusive: true },
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]'

// 复合形式
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  new Date(Date.UTC(2016, 1, 1)),
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

Timeline.create({ range });
```

但请注意,每当你收到一个范围值,你将收到:

```js
// 存储值: ("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]
range // [{ value: Date, inclusive: false }, { value: Date, inclusive: true }]
```

在使用 range 类型更新实例后,你需要调用 reload,或使用 `returning:true` 参数.

#### 特别案例

```js
// 空的范围:
Timeline.create({ range: [] }); // range = 'empty'

// 无边界范围:
Timeline.create({ range: [null, null] }); // range = '[,)'
// range = '[,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [null, new Date(Date.UTC(2016, 0, 1))] });

// 无穷范围:
// range = '[-infinity,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [-Infinity, new Date(Date.UTC(2016, 0, 1))] });
```

## 扩展数据类型

你尝试实现的类型很可能已包含在 [DataTypes](/manual/data-types.html) 中. 如果未包含新数据类型,本手册将说明如何自行编写.

Sequelize 不会在数据库中创建新的数据类型. 本教程介绍如何使 Sequelize 识别新的数据类型,并假设已在数据库中创建了这些新的数据类型.

要扩展 Sequelize 数据类型,请在创建任何实例之前执行此操作. 此示例创建一个虚拟的 `NEWTYPE`,它复制内置数据类型 `Sequelize.INTEGER(11).ZEROFILL.UNSIGNED`.

```js
// myproject/lib/sequelize.js

const Sequelize = require('Sequelize');
const sequelizeConfig = require('../config/sequelize')
const sequelizeAdditions = require('./sequelize-additions')

// 添加新数据类型的函数
sequelizeAdditions(Sequelize)

// 在这个例子中,创建并导出Sequelize实例
const sequelize = new Sequelize(sequelizeConfig)

module.exports = sequelize
```

```js
// myproject/lib/sequelize-additions.js

module.exports = function sequelizeAdditions(Sequelize) {

  DataTypes = Sequelize.DataTypes

  /*
   * 创建新类型
   */
  class NEWTYPE extends DataTypes.ABSTRACT {
    // 强制,在数据库中完整定义新类型
    toSql() {
      return 'INTEGER(11) UNSIGNED ZEROFILL'
    }

    // 可选,验证器功能
    validate(value, options) {
      return (typeof value === 'number') && (! Number.isNaN(value))
    }

    // 可选,sanitizer
    _sanitize(value) {
      // Force all numbers to be positive
      if (value < 0) {
        value = 0
      }

      return Math.round(value)
    }

    // 可选,发送到数据库之前的值字符串
    _stringify(value) {
      return value.toString()
    }

    // 可选,解析从数据库接收的值
    static parse(value) {
      return Number.parseInt(value)
    }
  }
  
  DataTypes.NEWTYPE = NEWTYPE;

  // 强制,设置 Key
  DataTypes.NEWTYPE.prototype.key = DataTypes.NEWTYPE.key = 'NEWTYPE'

  // 可选, 在stringifier后禁用转义. 不建议.
  // 警告:禁用针对SQL注入的Sequelize保护
  // DataTypes.NEWTYPE.escape = false

  // 为了简便`classToInvokable` 允许你使用没有 `new` 的数据类型
  Sequelize.NEWTYPE = Sequelize.Utils.classToInvokable(DataTypes.NEWTYPE)

}
```

创建此新数据类型后,你需要在每个数据库方言中映射此数据类型并进行一些调整.

## PostgreSQL

假设在 postgres 数据库中新数据类型的名称是`pg_new_type`. 该名称必须映射到`DataTypes.NEWTYPE`. 此外,还需要创建 postgres 特定的子数据类型.

```js
// myproject/lib/sequelize-additions.js

module.exports = function sequelizeAdditions(Sequelize) {

  DataTypes = Sequelize.DataTypes

  /*
   * 创建新类型
   */

  ...

  /*
   * 映射新类型
   */

  // 强制, 映射 postgres 数据类型名称
  DataTypes.NEWTYPE.types.postgres = ['pg_new_type']

  // 强制, 使用自己的 parse 方法创建 postgres 特定的子数据类型. 解析器将动态映射到 pg_new_type 的 OID.
  PgTypes = DataTypes.postgres

  PgTypes.NEWTYPE = function NEWTYPE() {
    if (!(this instanceof PgTypes.NEWTYPE)) return new PgTypes.NEWTYPE();
    DataTypes.NEWTYPE.apply(this, arguments);
  }
  inherits(PgTypes.NEWTYPE, DataTypes.NEWTYPE);

  // 强制, 创建,覆盖或重新分配postgres特定的解析器
  //PgTypes.NEWTYPE.parse = value => value;
  PgTypes.NEWTYPE.parse = DataTypes.NEWTYPE.parse;

  // 可选, 添加或覆盖 postgres 特定数据类型的方法
  // 比如 toSql, escape, validate, _stringify, _sanitize...

}
```

### 范围

在[postgres中定义](https://www.postgresql.org/docs/current/static/rangetypes.html#RANGETYPES-DEFINING)新的范围类型之后, 将它添加到Sequelize是微不足道的.

在这个例子中,postgres范围类型的名称是`newtype_range`,底层postgres数据类型的名称是`pg_new_type`. `subtypes`和`castTypes`的关键是Sequelize数据类型`DataTypes.NEWTYPE.key`的关键字,小写.

```js
// myproject/lib/sequelize-additions.js

module.exports = function sequelizeAdditions(Sequelize) {

  DataTypes = Sequelize.DataTypes

  /*
   * 创建新类型
   */

  ...

  /*
   * 映射新类型
   */

  ...

  /*
   * 添加范围支持
   */

  // 添加 postgresql 范围,newtype 来自 DataType.NEWTYPE.key,小写
  DataTypes.RANGE.types.postgres.subtypes.newtype = 'newtype_range';
  DataTypes.RANGE.types.postgres.castTypes.newtype = 'pg_new_type';

}
```

新范围可以在模型定义中用作 `Sequelize.RANGE(Sequelize.NEWTYPE)`或`DataTypes.RANGE(DataTypes.NEWTYPE)`.
