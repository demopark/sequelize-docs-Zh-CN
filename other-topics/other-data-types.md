# Other Data Types - 其他数据类型

Sequelize 提供了[很多内置数据类型](/api/v7/modules/datatypes). 要访问内置数据类型, 你必须导入 `DataTypes`:

```js
// 导入内置数据类型
import { DataTypes } from '@sequelize/core';
```

下面是一系列支持表, 描述了每种 Sequelize 数据类型使用哪种 SQL 类型.

**注意**

我们的大多数数据类型也接受参数包. 点击下表中的一种数据类型以查看其签名.

❌ 表示方言不支持该数据类型.

## Strings

<DialectTableFilter>

| Sequelize DataType                                                                 | PostgreSQL                                                      | MariaDB                            | MySQL                        | MSSQL                          | [SQLite][sqlite-datatypes] | [Snowflake](https://docs.snowflake.com/en/sql-reference/data-types-text.html) | db2                         | ibmi                        |
|------------------------------------------------------------------------------------|-----------------------------------------------------------------|------------------------------------|------------------------------|--------------------------------|----------------------------|-------------------------------------------------------------------------------|-----------------------------|-----------------------------|
| [`STRING`](pathname:///api/v7/interfaces/DataTypes.StringDataTypeConstructor.html) | [`VARCHAR(255)`][postgres-char]                                 | [`VARCHAR(255)`][mariadb-varchar]  | [`VARCHAR(255)`][mysql-char] | [`NVARCHAR(255)`][mssql-nchar] | `TEXT`                     | `VARCHAR(255)`                                                                | `VARCHAR(255)`              | `VARCHAR(255)`              |
| `STRING(100)`                                                                      | `VARCHAR(100)`                                                  | `VARCHAR(100)`                     | `VARCHAR(100)`               | `NVARCHAR(100)`                | `TEXT`                     | `VARCHAR(100)`                                                                | `VARCHAR(100)`              | `VARCHAR(100)`              |
| `STRING.BINARY`                                                                    | ❌                                                               | `VARCHAR(255) BINARY`              | `VARCHAR(255) BINARY`        | ❌                              | `TEXT COLLATE BINARY`      | `VARCHAR(255) BINARY`                                                         | `VARCHAR(255) FOR BIT DATA` | `VARCHAR(255) FOR BIT DATA` |
| `STRING(100).BINARY`                                                               | ❌                                                               | `VARCHAR(100) BINARY`              | `VARCHAR(100) BINARY`        | ❌                              | `TEXT COLLATE BINARY`      | `VARCHAR(100) BINARY`                                                         | `VARCHAR(100) FOR BIT DATA` | `VARCHAR(100) FOR BIT DATA` |
| [`TEXT`](pathname:///api/v7/interfaces/DataTypes.TextDataTypeConstructor.html)     | [`TEXT`][postgres-char]                                         | [`TEXT`][mariadb-text]             | [`TEXT`][mysql-text]         | `NVARCHAR(MAX)`                | `TEXT`                     | `TEXT`                                                                        | `CLOB(2147483647)`          | `CLOB(2147483647)`          |
| `TEXT('tiny')`                                                                     | `TEXT`                                                          | [`TINYTEXT`][mariadb-tinytext]     | [`TINYTEXT`][mysql-text]     | `NVARCHAR(256)`                | `TEXT`                     | `TEXT`                                                                        | `VARCHAR(256)`              | `VARCHAR(256)`              |
| `TEXT('medium')`                                                                   | `TEXT`                                                          | [`MEDIUMTEXT`][mariadb-mediumtext] | [`MEDIUMTEXT`][mysql-text]   | `NVARCHAR(MAX)`                | `TEXT`                     | `TEXT`                                                                        | `VARCHAR(16777216)`         | `VARCHAR(16777216)`         |
| `TEXT('long')`                                                                     | `TEXT`                                                          | [`LONGTEXT`][mariadb-longtext]     | [`LONGTEXT`][mysql-text]     | `NVARCHAR(MAX)`                | `TEXT`                     | `TEXT`                                                                        | `CLOB(2147483647)`          | `CLOB(2147483647)`          |
| [`CHAR`](pathname:///api/v7/interfaces/DataTypes.CharDataTypeConstructor.html)     | [`CHAR(255)`][postgres-char]                                    | [`CHAR(255)`][mariadb-char]        | [`CHAR(255)`][mysql-char]    | [`CHAR(255)`][mssql-char]      | ❌                          | `CHAR(255)`                                                                   | `CHAR(255)`                 | `CHAR(255)`                 |
| `CHAR(100)`                                                                        | `CHAR(100)`                                                     | `CHAR(100)`                        | `CHAR(100)`                  | `CHAR(100)`                    | ❌                          | `CHAR(100)`                                                                   | `CHAR(100)`                 | `CHAR(100)`                 |
| `CHAR.BINARY`                                                                      | ❌                                                               | `CHAR(255) BINARY`                 | `CHAR(255) BINARY`           | ❌                              | ❌                          | `CHAR(255) BINARY`                                                            | `CHAR(255) FOR BIT DATA`    | `CHAR(255) FOR BIT DATA`    |
| `CHAR(100).BINARY`                                                                 | ❌                                                               | `CHAR(100) BINARY`                 | `CHAR(100) BINARY`           | ❌                              | ❌                          | `CHAR(100) BINARY`                                                            | `CHAR(255) FOR BIT DATA`    | `CHAR(255) FOR BIT DATA`    |
| `CITEXT`                                                                           | [`CITEXT`](https://www.postgresql.org/docs/current/citext.html) | ❌                                  | ❌                            | ❌                              | `TEXT COLLATE NOCASE`      | ❌                                                                             | ❌                           | ❌                           |
| `TSVECTOR`                                                                         | [`TSVECTOR`][postgres-tsvector]                                 | ❌                                  | ❌                            | ❌                              | ❌                          | ❌                                                                             | ❌                           | ❌                           |

</DialectTableFilter>

## Boolean

<DialectTableFilter>

| Sequelize DataType | PostgreSQL                                                                 | MariaDB                         | MySQL                         | MSSQL                                                                                                 | SQLite                           | Snowflake                                                                        | db2       | ibmi       |
|--------------------|----------------------------------------------------------------------------|---------------------------------|-------------------------------|-------------------------------------------------------------------------------------------------------|----------------------------------|----------------------------------------------------------------------------------|-----------|------------|
| `BOOLEAN`          | [`BOOLEAN`](https://www.postgresql.org/docs/current/datatype-boolean.html) | [`TINYINT(1)`][mariadb-tinyint] | [`TINYINT(1)`][mysql-numeric] | [`BIT`](https://docs.microsoft.com/en-us/sql/t-sql/data-types/bit-transact-sql?view=sql-server-ver15) | [`TINYINT(1)`][sqlite-datatypes] | [`BOOLEAN`](https://docs.snowflake.com/en/sql-reference/data-types-logical.html) | `BOOLEAN` | `SMALLINT` |

</DialectTableFilter>

## Integers

<DialectTableFilter>

| Sequelize DataType                                                                           | [PostgreSQL][postgres-numeric] | [MariaDB][mariadb-numeric]                          | [MySQL][mysql-numeric] | [MSSQL][mssql-ints]  | [SQLite][sqlite-datatypes] | [Snowflake][snowflake-numeric] | db2                 | ibmi                |
|----------------------------------------------------------------------------------------------|--------------------------------|-----------------------------------------------------|------------------------|----------------------|----------------------------|--------------------------------|---------------------|---------------------|
| [`TINYINT`](pathname:///api/v7/interfaces/DataTypes.TinyIntegerDataTypeConstructor.html)     | `SMALLINT`[^ints-1]            | [`TINYINT`][mariadb-tinyint]                        | `TINYINT`              | `SMALLINT`[^mssql-1] | `INTEGER`                  | `INTEGER`                      | `SMALLINT`[^ints-1] | `SMALLINT`[^ints-1] |
| `TINYINT(1)`                                                                                 | ❌                              | `TINYINT(1)`                                        | `TINYINT(1)`           | ❌                    | ❌                          | `❌                             | ❌                   | ❌                   |
| `TINYINT.UNSIGNED`                                                                           | `SMALLINT`                     | `TINYINT UNSIGNED`                                  | `TINYINT UNSIGNED`     | `TINYINT`[^mssql-1]  | `INTEGER`                  | `INTEGER`                      | `SMALLINT`          | `SMALLINT`          |
| `TINYINT.ZEROFILL`                                                                           | ❌                              | `TINYINT ZEROFILL`                                  | `TINYINT ZEROFILL`     | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| [`SMALLINT`](pathname:///api/v7/interfaces/DataTypes.SmallIntegerDataTypeConstructor.html)   | `SMALLINT`                     | [`SMALLINT`](https://mariadb.com/kb/en/smallint/)   | `SMALLINT`             | `SMALLINT`           | `INTEGER`                  | `INTEGER`                      | `SMALLINT`          | `SMALLINT`          |
| `SMALLINT(1)`                                                                                | ❌                              | `SMALLINT(1)`                                       | `SMALLINT(1)`          | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| `SMALLINT.UNSIGNED`                                                                          | `INTEGER`[^ints-2]             | `SMALLINT UNSIGNED`                                 | `SMALLINT UNSIGNED`    | `INTEGER`[^ints-2]   | `INTEGER`                  | `INTEGER`                      | `INTEGER`[^ints-2]  | `INTEGER`[^ints-2]  |
| `SMALLINT.ZEROFILL`                                                                          | ❌                              | `SMALLINT ZEROFILL`                                 | `SMALLINT ZEROFILL`    | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| [`MEDIUMINT`](pathname:///api/v7/interfaces/DataTypes.MediumIntegerDataTypeConstructor.html) | `INTEGER`                      | [`MEDIUMINT`](https://mariadb.com/kb/en/mediumint/) | `MEDIUMINT`            | `INTEGER`[^ints-1]   | `INTEGER`                  | `INTEGER`                      | `INTEGER`           | `INTEGER`           |
| `MEDIUMINT(1)`                                                                               | ❌                              | `MEDIUMINT(1)`                                      | `MEDIUMINT(1)`         | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| `MEDIUMINT.UNSIGNED`                                                                         | `INTEGER`                      | `MEDIUMINT UNSIGNED`                                | `MEDIUMINT UNSIGNED`   | `INTEGER`            | `INTEGER`                  | `INTEGER`                      | `INTEGER`           | `INTEGER`           |
| `MEDIUMINT.ZEROFILL`                                                                         | ❌                              | `MEDIUMINT ZEROFILL`                                | `MEDIUMINT ZEROFILL`   | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| [`INTEGER`](pathname:///api/v7/interfaces/IntegerDataTypeConstructor.html)                   | `INTEGER`                      | [`INTEGER`](https://mariadb.com/kb/en/integer/)     | `INTEGER`              | `INTEGER`            | `INTEGER`                  | `INTEGER`                      | `INTEGER`           | `INTEGER`           |
| `INTEGER(1)`                                                                                 | ❌                              | `INTEGER(1)`                                        | `INTEGER(1)`           | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| `INTEGER.UNSIGNED`                                                                           | `BIGINT`                       | `INTEGER UNSIGNED`                                  | `INTEGER UNSIGNED`     | `BIGINT`             | `INTEGER`                  | `INTEGER`                      | `BIGINT`            | `BIGINT`            |
| `INTEGER.ZEROFILL`                                                                           | ❌                              | `INTEGER ZEROFILL`                                  | `INTEGER ZEROFILL`     | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| [`BIGINT`](pathname:///api/v7/interfaces/DataTypes.BigIntDataTypeConstructor.html)           | `BIGINT`                       | [`BIGINT`](https://mariadb.com/kb/en/bigint/)       | `BIGINT`               | `BIGINT`             | `INTEGER`                  | `INTEGER`                      | `BIGINT`            | `BIGINT`            |
| `BIGINT(1)`                                                                                  | ❌                              | `BIGINT(1)`                                         | `BIGINT(1)`            | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |
| `BIGINT.UNSIGNED`                                                                            | ❌                              | `BIGINT UNSIGNED`                                   | `BIGINT UNSIGNED`      | ❌                    | `INTEGER`                  | `INTEGER`                      | ❌                   | ❌                   |
| `BIGINT.ZEROFILL`                                                                            | ❌                              | `BIGINT ZEROFILL`                                   | `BIGINT ZEROFILL`      | ❌                    | ❌                          | ❌                              | ❌                   | ❌                   |

</DialectTableFilter>

[^ints-1]: 当 int 类型不可用时, Sequelize 使用更大的 int 类型.
[^ints-2]: 当一个 unsigned int 类型不可用时, Sequelize 使用一个更大的 int 类型来确保该类型覆盖所有可能的较小 int 类型的无符号整数值. 
[^mssql-1]: SQL Server 中的 `TINYINT` 是未签名的.  因此, `DataTypes.TINYINT.UNSIGNED` 映射到 `TINYINT`, 而 `DataTypes.TINYINT` 映射到 `SMALLINT`. 

**注意**

JavaScript [`number`][mdn-number] 类型可以表示范围从  [`-9007199254740991`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MIN_SAFE_INTEGER) 
到 [`9007199254740991`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER).

如果你的 SQL 类型支持超出此范围的整数值, 我们建议使用 [`bigint`][mdn-bigint] 或 [`string`][mdn-string] 来表示你的整数.

可以组合数字参数:

`DataTypes.INTEGER(1).UNSIGNED.ZEROFILL` 将在 MySQL 中生成类型为 `INTEGER(1) UNSIGNED ZEROFILL` 的列.

## 近似小数数字

下表中的类型通常表示为 [IEEE 754 浮点数][ieee-754], 如 JavaScript [`number`][mdn-number] 类型.

- `FLOAT` 是一种单精度浮点类型.
- `DOUBLE` 是双精度浮点类型.

<DialectTableFilter>

| Sequelize DataType                                                                 | [PostgreSQL][postgres-numeric] | [MariaDB][mariadb-numeric]                              | [MySQL][mysql-numeric]      | [MSSQL][mssql-inexact-decimals] | [SQLite][sqlite-datatypes] | [Snowflake][snowflake-numeric] | db2      | ibmi     |
|------------------------------------------------------------------------------------|--------------------------------|---------------------------------------------------------|-----------------------------|---------------------------------|----------------------------|--------------------------------|----------|----------|
| [`FLOAT`](pathname:///api/v7/interfaces/DataTypes.FloatDataTypeConstructor.html)   | `REAL`                         | [`FLOAT`](https://mariadb.com/kb/en/float/)             | `FLOAT`                     | `REAL`                          | `REAL`[^sqlite-3]          | `FLOAT`[^snowflake-1]          | `REAL`   | `REAL`   |
| `FLOAT(11, 10)`                                                                    | ❌                              | `FLOAT(11,10)`                                          | `FLOAT(11,10)`              | ❌                               | ❌                          | ❌                              | ❌        | ❌        |
| `FLOAT.UNSIGNED`                                                                   | `REAL`                         | `FLOAT UNSIGNED`                                        | `FLOAT UNSIGNED`            | `REAL`                          | `REAL`                     | `FLOAT`                        | `REAL`   | `REAL`   |
| `FLOAT.ZEROFILL`                                                                   | ❌                              | `FLOAT ZEROFILL`                                        | `FLOAT ZEROFILL`            | ❌                               | ❌                          | ❌                              | ❌        | ❌        |
| [`DOUBLE`](pathname:///api/v7/interfaces/DataTypes.DoubleDataTypeConstructor.html) | `DOUBLE PRECISION`             | [`DOUBLE PRECISION`](https://mariadb.com/kb/en/double/) | `DOUBLE PRECISION`          | `DOUBLE PRECISION`              | `REAL`                     | `FLOAT`                        | `DOUBLE` | `DOUBLE` |
| `DOUBLE(11, 10)`                                                                   | ❌                              | `DOUBLE PRECISION(11, 10)`                              | `DOUBLE PRECISION(11, 10)`  | ❌                               | ❌                          | ❌                              | ❌        | ❌        |
| `DOUBLE.UNSIGNED`                                                                  | `DOUBLE PRECISION`             | `DOUBLE PRECISION UNSIGNED`                             | `DOUBLE PRECISION UNSIGNED` | `DOUBLE PRECISION`              | `REAL`                     | `FLOAT`                        | `DOUBLE` | `DOUBLE` |
| `DOUBLE.ZEROFILL`                                                                  | ❌                              | `DOUBLE PRECISION ZEROFILL`                             | `DOUBLE PRECISION ZEROFILL` | ❌                               | ❌                          | ❌                              | ❌        | ❌        |

</DialectTableFilter>

[^sqlite-3]: 与其他方言不同, 在 SQLite 中, `REAL` 是一种双精度浮点数类型. 
[^snowflake-1]: 与其他方言不同, 在 Snowflake 中, `FLOAT` 是一种双精度浮点数类型. 

**注意**

可以组合数字参数:

`DataTypes.FLOAT(1, 2).UNSIGNED.ZEROFILL` 将在 MySQL 中生成类型为 `FLOAT(1, 2) UNSIGNED ZEROFILL` 的列.

## 精确小数数字

- `DECIMAL` 是一种不受约束的十进制类型.
- `DECIMAL(precision, scale)` 是一种受约束的小数类型.

<DialectTableFilter>

| Sequelize DataType                                                                   | [PostgreSQL][postgres-numeric] | [MariaDB][mariadb-numeric]                             | [MySQL][mysql-numeric]   | [MSSQL][mssql-exact-decimals] | [SQLite][sqlite-datatypes] | [Snowflake][snowflake-numeric] | db2              | ibmi             |
|--------------------------------------------------------------------------------------|--------------------------------|--------------------------------------------------------|--------------------------|-------------------------------|----------------------------|--------------------------------|------------------|------------------|
| [`DECIMAL`](pathname:///api/v7/interfaces/DataTypes.DecimalDataTypeConstructor.html) | `DECIMAL`                      | ❌                                                      | ❌                        | ❌                             | ❌                          | ❌                              | ❌                | ❌                |
| `DECIMAL(11, 10)`                                                                    | `DECIMAL(11, 10)`              | [`DECIMAL(11,10)`](https://mariadb.com/kb/en/decimal/) | `DECIMAL(11,10)`         | `DECIMAL(11,10)`              | ❌                          | `DECIMAL(11,10)`               | `DECIMAL(11,10)` | `DECIMAL(11,10)` |
| `DECIMAL(p, s).UNSIGNED`                                                             | `DECIMAL(p, s)`                | `DECIMAL(p, s) UNSIGNED`                               | `DECIMAL(p, s) UNSIGNED` | `DECIMAL(p, s)`               | ❌                          | `DECIMAL(p, s)`                | `DECIMAL(p, s)`  | `DECIMAL(p, s)`  |
| `DECIMAL(p, s).ZEROFILL`                                                             | ❌                              | `DECIMAL(p, s) ZEROFILL`                               | `DECIMAL(p, s) ZEROFILL` | ❌                             | ❌                          | ❌                              | ❌                | ❌                |

</DialectTableFilter>

**注意**

精确的十进制数在 JavaScript 中[还](https://github.com/tc39/proposal-decimal)无法表示. 
JavaScript [`number`][mdn-number] 类型是双精度 64 位二进制格式 [IEEE 754][ieee-754] 值, 更好地表示 [不精确的小数类型].

为避免任何精度损失, 我们建议使用 [`string`][mdn-string] 在 JavaScript 中表示精确的小数. 

可以组合数字参数:

`DataTypes.DECIMAL(1, 2).UNSIGNED.ZEROFILL` 将在 MySQL 中生成类型为 `DECIMAL(1, 2) UNSIGNED ZEROFILL` 的列.

## Dates

<DialectTableFilter>

| Sequelize DataType                                                             | [PostgreSQL][postgres-temporal] | [MariaDB][mariadb-temporal]                       | [MySQL][mysql-temporal]                                             | MSSQL                                                                                                 | SQLite | [Snowflake][snowflake-temporal] | db2            | ibmi           |
|--------------------------------------------------------------------------------|---------------------------------|---------------------------------------------------|---------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|--------|---------------------------------|----------------|----------------|
| [`DATE`](pathname:///api/v7/interfaces/DataTypes.DateDataTypeConstructor.html) | `TIMESTAMP WITH TIME ZONE`      | [`DATETIME`](https://mariadb.com/kb/en/datetime/) | [`DATETIME`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html) | [`DATETIMEOFFSET`](https://docs.microsoft.com/en-us/sql/t-sql/data-types/datetimeoffset-transact-sql) | `TEXT` | `TIMESTAMP`                     | `TIMESTAMP`    | `TIMESTAMP`    |
| `DATE(6)`                                                                      | `TIMESTAMP(6) WITH TIME ZONE`   | `DATETIME(6)`                                     | `DATETIME(6)`                                                       | `DATETIMEOFFSET(6)`                                                                                   | `TEXT` | `TIMESTAMP(6)`                  | `TIMESTAMP(6)` | `TIMESTAMP(6)` |
| `DATEONLY`                                                                     | `DATE`                          | [`DATE`](https://mariadb.com/kb/en/date/)         | [`DATE`](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)     | [`DATE`](https://docs.microsoft.com/en-us/sql/t-sql/data-types/date-transact-sql)                     | `TEXT` | `DATE`                          | `DATE`         | `DATE`         |
| `TIME`                                                                         | `TIME`                          | [`TIME`](https://mariadb.com/kb/en/time/)         | [`TIME`](https://dev.mysql.com/doc/refman/8.0/en/time.html)         | [`TIME`](https://docs.microsoft.com/en-us/sql/t-sql/data-types/time-transact-sql)                     | `TEXT` | `TIME`                          | `TIME`         | `TIME`         |
| `TIME(6)`                                                                      | `TIME(6)`                       | `TIME(6)`                                         | `TIME(6)`                                                           | `TIME(6)`                                                                                             | `TEXT` | `TIME(6)`                       | `TIME(6)`      | `TIME(6)`      |

</DialectTableFilter>

### 日期的内置默认值

除了常规 [默认值](../core-concepts/model-basics.md), Sequelize 还提供了 `DataTypes.NOW` 它将根据你的方言使用适当的原生 SQL 函数.

<DialectTableFilter>

| Sequelize DataType | PostgreSQL                                                           | MariaDB                                 | MySQL                                                                         | MSSQL                                                                                                          | SQLite | Snowflake | db2            | ibmi  |
|--------------------|----------------------------------------------------------------------|-----------------------------------------|-------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|--------|-----------|----------------|-------|
| `NOW`              | [`NOW`](https://www.postgresql.org/docs/8.2/functions-datetime.html) | [`NOW`](https://mariadb.com/kb/en/now/) | [`NOW`](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html) | [`GETDATE()`](https://docs.microsoft.com/en-us/sql/t-sql/functions/getdate-transact-sql?view=sql-server-ver15) | ❌      | `NOW`     | `CURRENT TIME` | `NOW` |

</DialectTableFilter>

```javascript
MyModel.init({
  myDate: {
    type: DataTypes.DATE,
    defaultValue: DataTypes.NOW,
  },
});
```

## UUIDs

对于 UUIDs, use `DataTypes.UUID`.  它将成为 PostgreSQL 和 SQLite 的 `UUID` 数据类型, 以及 MySQL 的 `CHAR(36)`.

<DialectTableFilter>

| Sequelize DataType | PostgreSQL                                                           | MariaDB           | MySQL             | MSSQL                                                                                                                            | SQLite | Snowflake     | db2                     | ibmi                    |
|--------------------|----------------------------------------------------------------------|-------------------|-------------------|----------------------------------------------------------------------------------------------------------------------------------|--------|---------------|-------------------------|-------------------------|
| `UUID`             | [`UUID`](https://www.postgresql.org/docs/current/datatype-uuid.html) | `CHAR(36) BINARY` | `CHAR(36) BINARY` | [`UNIQUEIDENTIFIER`](https://learn.microsoft.com/en-us/sql/t-sql/data-types/uniqueidentifier-transact-sql?view=sql-server-ver16) | `TEXT` | `VARCHAR(36)` | `CHAR(36) FOR BIT DATA` | `CHAR(36) FOR BIT DATA` |

</DialectTableFilter>

### UUID 的内置默认值

Sequelize 可以为这些字段自动生成 UUID, 只需使用 `DataTypes.UUIDV1` 或 `DataTypes.UUIDV4` 作为默认值:

```javascript
MyModel.init({
  myUuid: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4, // Or DataTypes.UUIDV1
  },
});
```

**注意**

`DataTypes.UUIDV1` 和 `DataTypes.UUIDV4` 的值的生成由 JavaScript 中的 Sequelize 层完成. 
因此, 它仅在与模型交互时使用.  它不能用于 [migrations](../other-topics/migrations.md).

如果你的方言提供了一个内置的 SQL 函数来生成 UUID, 你可以使用 `fn` 在 SQL 层上设置一个默认值. 
使其可用于原始查询和迁移. 

```javascript
import { fn } from '@sequelize/core';

MyModel.init({
  myUuid: {
    type: DataTypes.UUID,
    // 'uuid_generate_v4' 仅在 postgres + uuid-ossp 可用
    // 其他方言可能以不同的名称支持此功能.
    defaultValue: fn('uuid_generate_v4'),
  },
});
```

:::

## BLOBs

blob 数据类型允许你以字符串和缓冲区的形式插入数据. 但是, 当使用 Sequelize 从数据库中检索 blob 时, 它将始终作为 [Node Buffer](https://nodejs.org/api/buffer.html) 检索.

<DialectTableFilter>

| Sequelize DataType                                                             | PostgreSQL | MariaDB      | MySQL        | MSSQL            | [SQLite][sqlite-datatypes] | Snowflake    | db2         | ibmi        |
|--------------------------------------------------------------------------------|------------|--------------|--------------|------------------|----------------------------|--------------|-------------|-------------|
| [`BLOB`](pathname:///api/v7/interfaces/DataTypes.BlobDataTypeConstructor.html) | `BYTEA`    | `BLOB`       | `BLOB`       | `VARBINARY(MAX)` | `BLOB`                     | `BLOB`       | `BLOB(1M)`  | `BLOB(1M)`  |
| `BLOB('tiny')`                                                                 | `BYTEA`    | `TINYBLOB`   | `TINYBLOB`   | `VARBINARY(256)` | `BLOB`                     | `TINYBLOB`   | `BLOB(255)` | `BLOB(255)` |
| `BLOB('medium')`                                                               | `BYTEA`    | `MEDIUMBLOB` | `MEDIUMBLOB` | `VARBINARY(MAX)` | `BLOB`                     | `MEDIUMBLOB` | `BLOB(16M)` | `BLOB(16M)` |
| `BLOB('long')`                                                                 | `BYTEA`    | `LONGBLOB`   | `LONGBLOB`   | `VARBINARY(MAX)` | `BLOB`                     | `LONGBLOB`   | `BLOB(2G)`  | `BLOB(2G)`  |

</DialectTableFilter>

## ENUMs

**注意**

Enums 仅在 [PostgreSQL](https://www.postgresql.org/docs/current/datatype-enum.html), [MariaDB](https://mariadb.com/kb/en/enum/), 和 [MySQL](https://dev.mysql.com/doc/refman/8.0/en/enum.html) 中可用.

ENUM 是一种只接受少数值的数据类型, 指定为列表.

```js
DataTypes.ENUM('foo', 'bar') // 具有允许值 'foo' 和 'bar' 的 ENUM
```

有关此 DataType 接受的参数的更多信息, 请参阅 [DataTypes.ENUM 的 API 参考](/api/v7/interfaces/DataTypes.EnumDataTypeConstructor.html).

## JSON & JSONB

`DataTypes.JSON` 数据类型仅支持 SQLite, MySQL, MariaDB 和 PostgreSQL. 但是, 对 MSSQL 有最低限度的支持(见下文).

<DialectTableFilter>

| Sequelize DataType | PostgreSQL                                                        | MariaDB                                             | MySQL                                                       | MSSQL           | [SQLite][sqlite-datatypes] | Snowflake | db2 | ibmi |
|--------------------|-------------------------------------------------------------------|-----------------------------------------------------|-------------------------------------------------------------|-----------------|----------------------------|-----------|-----|------|
| `JSON`             | [`JSON`](https://www.postgresql.org/docs/9.4/datatype-json.html)  | [`JSON`](https://mariadb.com/kb/en/json-data-type/) | [`JSON`](https://dev.mysql.com/doc/refman/8.0/en/json.html) | `NVARCHAR(MAX)` | `TEXT`                     | ❌         | ❌   | ❌    |
| `JSONB`            | [`JSONB`](https://www.postgresql.org/docs/9.4/datatype-json.html) | ❌                                                   | ❌                                                           | ❌               | ❌                          | ❌         | ❌   | ❌    |

</DialectTableFilter>

PostgreSQL 中的 JSON 数据类型将值存储为纯文本, 而不是二进制表示.

如果你只是想存储和检索 JSON 表示, 使用 JSON 将占用更少的磁盘空间和更少的时间来构建其输入表示.  但是, 如果你想对 JSON 值进行任何操作, 你应该首选 JSONB 数据类型. 

**查询 JSON**

Sequelize 提供了一种特殊的语法来查询 JSON 对象的内容.  [阅读有关查询 JSON 的更多信息](../core-concepts/model-querying-basics.md#querying-json). 

## 杂项数据类型

<DialectTableFilter>

| Sequelize DataType                                                                       | PostgreSQL                                                                | [MariaDB](https://mariadb.com/kb/en/geometry-types/) | [MySQL](https://dev.mysql.com/doc/refman/8.0/en/spatial-type-overview.html) | MSSQL | SQLite | Snowflake | db2 | ibmi |
|------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|------------------------------------------------------|-----------------------------------------------------------------------------|-------|--------|-----------|-----|------|
| [`GEOMETRY`](pathname:///api/v7/interfaces/DataTypes.GeometryDataTypeConstructor.html)   | [`GEOMETRY`](https://postgis.net/workshops/postgis-intro/geometries.html) | `GEOMETRY`                                           | `GEOMETRY`                                                                  | ❌     | ❌      | ❌         | ❌   | ❌    |
| `GEOMETRY('POINT')`                                                                      | `GEOMETRY(POINT)`                                                         | `POINT`                                              | `POINT`                                                                     | ❌     | ❌      | ❌         | ❌   | ❌    |
| `GEOMETRY('POINT', 4326)`                                                                | `GEOMETRY(POINT,4326)`                                                    | ❌                                                    | ❌                                                                           | ❌     | ❌      | ❌         | ❌   | ❌    |
| `GEOMETRY('POLYGON')`                                                                    | `GEOMETRY(POLYGON)`                                                       | `POLYGON`                                            | `POLYGON`                                                                   | ❌     | ❌      | ❌         | ❌   | ❌    |
| `GEOMETRY('LINESTRING')`                                                                 | `GEOMETRY(LINESTRING)`                                                    | `LINESTRING`                                         | `LINESTRING`                                                                | ❌     | ❌      | ❌         | ❌   | ❌    |
| [`GEOGRAPHY`](pathname:///api/v7/interfaces/DataTypes.GeographyDataTypeConstructor.html) | [`GEOGRAPHY`](https://postgis.net/workshops/postgis-intro/geography.html) | ❌                                                    | ❌                                                                           | ❌     | ❌      | ❌         | ❌   | ❌    |
| `HSTORE`                                                                                 | [`HSTORE`](https://www.postgresql.org/docs/9.1/hstore.html)               | ❌                                                    | ❌                                                                           | ❌     | ❌      | ❌         | ❌   | ❌    |

</DialectTableFilter>

**注意**

在 Postgres 中, GEOMETRY 和 GEOGRAPHY 类型由 [PostGIS 扩展](https://postgis.net/workshops/postgis-intro/geometries.html) 实现. 

在 Postgres 中, 如果使用 `DataTypes.HSTORE`, 则必须安装 [pg-hstore](https://www.npmjs.com/package/pg-hstore) 包.

## PostgreSQL 独有的数据类型

### Arrays

**注意**

[Arrays](https://www.postgresql.org/docs/current/arrays.html) 仅在 PostgreSQL 中可用.

```typescript
// 定义 DataTypes.SOMETHING 的数组
DataTypes.ARRAY(/* DataTypes.SOMETHING */)

// VARCHAR(255)[]
DataTypes.ARRAY(DataTypes.STRING)

// VARCHAR(255)[][]
DataTypes.ARRAY(DataTypes.ARRAY(DataTypes.STRING))
```

### Ranges

**注意**

[Ranges](https://www.postgresql.org/docs/current/rangetypes.html) 仅在 PostgreSQL 中可用.

```js
DataTypes.RANGE(DataTypes.INTEGER)    // int4range
DataTypes.RANGE(DataTypes.BIGINT)     // int8range
DataTypes.RANGE(DataTypes.DATE)       // tstzrange
DataTypes.RANGE(DataTypes.DATEONLY)   // daterange
DataTypes.RANGE(DataTypes.DECIMAL)    // numrange
```

由于范围类型有关于它们的绑定包含/排除的额外信息, 因此仅使用元组在 javascript 中表示它们并不是很简单. 

当提供范围作为值时, 你可以从以下 API 中进行选择:

```js
// 默认为包含下限, 不包含上限
const range = [
  new Date(Date.UTC(2016, 0, 1)),
  new Date(Date.UTC(2016, 1, 1))
];
// '["2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

// 包含控制
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

然而, 检索到的范围值总是以对象数组的形式出现.  例如, 如果存储的值为`("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]`, 在finder查询之后你会得到：

```js
[
  { value: Date, inclusive: false },
  { value: Date, inclusive: true }
]
```

你需要在使用范围类型更新实例后调用 `reload()`或使用 `returning: true` 参数.

#### 特殊情况

```js
// 空 range:
Timeline.create({ range: [] }); // range = 'empty'

// 无界限 range:
Timeline.create({ range: [null, null] }); // range = '[,)'
// range = '[,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [null, new Date(Date.UTC(2016, 0, 1))] });

// 无穷 range:
// range = '[-infinity,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [-Infinity, new Date(Date.UTC(2016, 0, 1))] });
```

#### TypeScript

使用 Sequelize 提供的 `Range` 类型来正确输入你的范围:

```typescript
import { Model, InferAttributes, Range } from '@sequelize/core';

class User extends Model<InferAttributes<User>> {
  declare myDateRange: Range<Date>;
}

User.init({
  myDateRange: {
    type: DataTypes.RANGE(DataTypes.DATE),
    allowNull: false,
  }
});
```

### 网络地址

<DialectTableFilter>

| Sequelize DataType | PostgreSQL                                                               | MariaDB | MySQL | MSSQL | SQLite | Snowflake | db2 | ibmi |
|--------------------|--------------------------------------------------------------------------|---------|-------|-------|--------|-----------|-----|------|
| `CIDR`             | [`CIDR`](https://www.postgresql.org/docs/9.1/datatype-net-types.html)    | ❌       | ❌     | ❌     | ❌      | ❌         | ❌   | ❌    |
| `INET`             | [`INET`](https://www.postgresql.org/docs/9.1/datatype-net-types.html)    | ❌       | ❌     | ❌     | ❌      | ❌         | ❌   | ❌    |
| `MACADDR`          | [`MACADDR`](https://www.postgresql.org/docs/9.1/datatype-net-types.html) | ❌       | ❌     | ❌     | ❌      | ❌         | ❌   | ❌    |

</DialectTableFilter>

## Virtual

`DataTypes.VIRTUAL` 是一种特殊的数据类型, 用于声明[虚拟属性](../core-concepts/getters-setters-virtuals.md#virtual-fields). 
它不会创建实际的列. 

**注意**

与 `GENERATED` 列不同, `DataTypes.VIRTUAL` 列在 JavaScript 层中处理.  它们不是在数据库表上创建的. 
请参阅[有关生成列的问题](https://github.com/sequelize/sequelize/issues/12718) 了解更多信息. 

## 自定义数据类型

数据库支持更多 Sequelize 中内置的数据类型未涵盖的数据类型. 
如果你需要使用这样的数据类型, 你可以[创建自己的数据类型](../other-topics/extending-data-types.md). 

也可以使用原始 SQL 字符串作为属性的类型. 创建表时, 此字符串将按原样用作列的类型. 

```typescript
User = sequelize.define('user', {
  password: {
    type: 'VARBINARY(50)',
  },
});
```

**注意:** Sequelize 不会对这样声明的属性进行任何额外的类型转换或验证.  请谨慎地使用！

当然, 你可以在 [Sequelize 存储库](https://github.com/sequelize/sequelize) 中打开功能请求
请求添加新的内置数据类型.

[mdn-number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number
[mdn-bigint]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt
[mdn-string]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String

[ieee-754]: https://en.wikipedia.org/wiki/IEEE_754

[postgres-char]: https://www.postgresql.org/docs/current/datatype-character.html
[postgres-binary]: https://www.postgresql.org/docs/9.0/datatype-binary.html
[postgres-tsvector]: https://www.postgresql.org/docs/10/datatype-textsearch.html
[postgres-numeric]: https://www.postgresql.org/docs/current/datatype-numeric.html
[postgres-temporal]: https://www.postgresql.org/docs/current/datatype-datetime.html

[mariadb-varchar]: https://mariadb.com/kb/en/varchar/
[mariadb-char]: https://mariadb.com/kb/en/char/
[mariadb-text]: https://mariadb.com/kb/en/text/
[mariadb-tinytext]: https://mariadb.com/kb/en/tinytext/
[mariadb-mediumtext]: https://mariadb.com/kb/en/mediumtext/
[mariadb-longtext]: https://mariadb.com/kb/en/longtext/
[mariadb-numeric]: https://mariadb.com/kb/en/data-types-numeric-data-types/
[mariadb-tinyint]: https://mariadb.com/kb/en/tinyint/
[mariadb-temporal]: https://mariadb.com/kb/en/date-and-time-data-types/

[mysql-char]: https://dev.mysql.com/doc/refman/8.0/en/char.html
[mysql-text]: https://dev.mysql.com/doc/refman/8.0/en/blob.html
[mysql-numeric]: https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html
[mysql-temporal]: https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html

[mssql-char]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/char-and-varchar-transact-sql?view=sql-server-ver15
[mssql-nchar]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql?view=sql-server-ver15
[mssql-text]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=sql-server-ver15
[mssql-binary]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/binary-and-varbinary-transact-sql?view=sql-server-ver15
[mssql-ints]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/int-bigint-smallint-and-tinyint-transact-sql?view=sql-server-ver15
[mssql-exact-decimals]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/decimal-and-numeric-transact-sql?view=sql-server-ver15
[mssql-inexact-decimals]: https://docs.microsoft.com/en-us/sql/t-sql/data-types/float-and-real-transact-sql?view=sql-server-ver15

[sqlite-datatypes]: https://www.sqlite.org/stricttables.html

[snowflake-numeric]: https://docs.snowflake.com/en/sql-reference/data-types-numeric.html
[snowflake-temporal]: https://docs.snowflake.com/en/sql-reference/data-types-datetime.html
