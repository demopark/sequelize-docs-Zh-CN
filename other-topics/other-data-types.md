# Other Data Types - 其他数据类型

除了[模型基础](core-concepts/model-basics.md)指南中提到的最常见的数据类型外,Sequelize 还提供了其他几种数据类型.

## 范围 (仅限 PostgreSQL)

```js
DataTypes.RANGE(DataTypes.INTEGER)    // int4range
DataTypes.RANGE(DataTypes.BIGINT)     // int8range
DataTypes.RANGE(DataTypes.DATE)       // tstzrange
DataTypes.RANGE(DataTypes.DATEONLY)   // daterange
DataTypes.RANGE(DataTypes.DECIMAL)    // numrange
```

由于范围类型对于它们绑定的 包含/排除 具有额外的信息,因此仅使用元组在 javascript 中表示它们并不是很容易.

当提供范围值时,可以从以下 API 中进行选择：

```js
// 默认为包含下限,排除上限
const range = [
  new Date(Date.UTC(2016, 0, 1)),
  new Date(Date.UTC(2016, 1, 1))
];
// '["2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

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

const Timeline = sequelize.define('Timeline', {
  range: DataTypes.RANGE(DataTypes.DATE)
});

await Timeline.create({ range });
```

然而,检索到的范围值始终以对象数组的形式出现. 例如,如果在 finder 查询后,存储的值是 `("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]` 你会得到：

```js
[
  { value: Date, inclusive: false },
  { value: Date, inclusive: true }
]
```

使用范围类型更新实例后,你需要调用 `reload()` 或使用 `returning: true` 参数.

### 特别案例

```js
// 空范围:
Timeline.create({ range: [] }); // range = 'empty'

// 无界范围:
Timeline.create({ range: [null, null] }); // range = '[,)'
// range = '[,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [null, new Date(Date.UTC(2016, 0, 1))] });

// 无限范围:
// range = '[-infinity,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [-Infinity, new Date(Date.UTC(2016, 0, 1))] });
```

## BLOB

```js
DataTypes.BLOB                // BLOB (PostgreSQL 的 bytea)
DataTypes.BLOB('tiny')        // TINYBLOB (PostgreSQL 的 bytea)
DataTypes.BLOB('medium')      // MEDIUMBLOB (PostgreSQL 的 bytea)
DataTypes.BLOB('long')        // LONGBLOB (PostgreSQL 的 bytea)
```

Blob 数据类型允许你将数据既作为字符串又作为缓冲区插入. 但是,当使用 Sequelize从 数据库中检索 Blob 时,将始终将其作为缓冲区检索.

## ENUM

ENUM 是仅接受几个值(指定为列表)的数据类型.

```js
DataTypes.ENUM('foo', 'bar') // 允许值为'foo'和'bar'的ENUM
```

也可以使用列定义的 `values` 字段指定 ENUM,如下所示：

```js
sequelize.define('foo', {
  states: {
    type: DataTypes.ENUM,
    values: ['active', 'pending', 'deleted']
  }
});
```

## JSON (仅限 SQLite, MySQL, MariaDB 和 PostgreSQL)

仅 SQLite,MySQL,MariaDB 和 PostgreSQL 支持 `DataTypes.JSON` 数据类型. 但是,对 MSSQL 的支持最少(请参见下文).

### PostgreSQL 的注意事项

PostgreSQL 中的 JSON 数据类型将值存储为纯文本,而不是二进制表示. 如果只想存储和检索 JSON 表示形式,则使用 JSON 将占用更少的磁盘空间,并需要更少的时间从其输入表示形式进行构建. 但是,如果要对 JSON 值执行任何操作,则应首选以下所述的 JSONB 数据类型.

### JSONB (仅限 PostgreSQL)

PostgreSQL 还支持 JSONB 数据类型： `DataTypes.JSONB`. 可以通过三种不同的方式查询它：

```js
// 嵌套对象
await Foo.findOne({
  where: {
    meta: {
      video: {
        url: {
          [Op.ne]: null
        }
      }
    }
  }
});

// 嵌套键
await Foo.findOne({
  where: {
    "meta.audio.length": {
      [Op.gt]: 20
    }
  }
});

// 包含限制
await Foo.findOne({
  where: {
    meta: {
      [Op.contains]: {
        site: {
          url: 'http://google.com'
        }
      }
    }
  }
});
```

### MSSQL

MSSQL 没有 JSON 数据类型,但是自 SQL Server 2016 起,它确实通过某些函数提供了对以字符串形式存储的 JSON 的支持.使用这些函数,你将能够查询存储在字符串中的 JSON,但是所有返回的值将需要单独解析.

```js
// ISJSON - 测试字符串是否包含有效的 JSON
await User.findAll({
  where: sequelize.where(sequelize.fn('ISJSON', sequelize.col('userDetails')), 1)
})

// JSON_VALUE - 从 JSON 字符串中提取标量值
await User.findAll({
  attributes: [[ sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), 'address line 1']]
})

// JSON_VALUE - 从 JSON 字符串查询标量值
await User.findAll({
  where: sequelize.where(sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), '14, Foo Street')
})

// JSON_QUERY - 提取对象或数组
await User.findAll({
  attributes: [[ sequelize.fn('JSON_QUERY', sequelize.col('userDetails'), '$.address'), 'full address']]
})
```

## 其他

```js
DataTypes.ARRAY(/* DataTypes.SOMETHING */)  // 定义一个 DataTypes.SOMETHING 数组. 仅限 PostgreSQL.

DataTypes.CIDR                        // CIDR                  仅限 PostgreSQL
DataTypes.INET                        // INET                  仅限 PostgreSQL
DataTypes.MACADDR                     // MACADDR               仅限 PostgreSQL

DataTypes.GEOMETRY                    // 空间列. 仅限 PostgreSQL (使用 PostGIS) 或 MySQL.
DataTypes.GEOMETRY('POINT')           // 具有几何类型的空间列.仅限 PostgreSQL (使用 PostGIS) 或 MySQL.
DataTypes.GEOMETRY('POINT', 4326)     // 具有几何类型和 SRID 的空间列.仅限 PostgreSQL (使用 PostGIS) 或 MySQL.
```