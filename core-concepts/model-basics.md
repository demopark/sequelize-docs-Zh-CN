# Model Basics - 模型基础

在本教程中,你将学习 Sequelize 中的模型以及如何使用它们.

## 概念

模型是 Sequelize 的本质. 模型是代表数据库中表的抽象. 在 Sequelize 中,它是一个 [Model](https://sequelize.org/master/class/lib/model.js~Model.html) 的扩展类.

该模型告诉 Sequelize 有关它代表的实体的几件事,例如数据库中表的名称以及它具有的列(及其数据类型).

Sequelize 中的模型有一个名称. 此名称不必与它在数据库中表示的表的名称相同. 通常,模型具有单数名称(例如,`User`),而表具有复数名称(例如, `Users`),当然这是完全可配置的.

## 模型定义

在 Sequelize 中可以用两种等效的方式定义模型：

* 调用 [`sequelize.define(modelName, attributes, options)`](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-define)
* 扩展 [Model](https://sequelize.org/master/class/lib/model.js~Model.html) 并调用 [`init(attributes, options)`](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-init)

定义模型后,可通过其模型名称在 `sequelize.models` 中使用该模型.

为了学习一个示例,我们将考虑创建一个代表用户的模型,该模型具有一个 `firstName` 和一个 `lastName`. 我们希望将模型称为 `User`,并将其表示的表在数据库中称为 `Users`.

定义该模型的两种方法如下所示. 定义后,我们可以使用 `sequelize.models.User` 访问模型.

### 使用 [`sequelize.define`](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-define):

```js
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  // 在这里定义模型属性
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull 默认为 true
  }
}, {
  // 这是其他模型参数
});

// `sequelize.define` 会返回模型
console.log(User === sequelize.models.User); // true
```

### 扩展 [Model](https://sequelize.org/master/class/lib/model.js~Model.html)

```js
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory');

class User extends Model {}

User.init({
  // 在这里定义模型属性
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull 默认为 true
  }
}, {
  // 这是其他模型参数
  sequelize, // 我们需要传递连接实例
  modelName: 'User' // 我们需要选择模型名称
});

// 定义的模型是类本身
console.log(User === sequelize.models.User); // true
```

在内部,`sequelize.define` 调用 `Model.init`,因此两种方法本质上是等效的.

## 表名推断

请注意,在以上两种方法中,都从未明确定义表名(`Users`). 但是,给出了模型名称(`User`).

默认情况下,当未提供表名时,Sequelize 会自动将模型名复数并将其用作表名. 这种复数是通过称为 [inflection](https://www.npmjs.com/package/inflection) 的库在后台完成的,因此可以正确计算不规则的复数(例如 `person -> people`).

当然,此行为很容易配置.

### 强制表名称等于模型名称

你可以使用 `freezeTableName: true` 参数停止 Sequelize 执行自动复数化. 这样,Sequelize 将推断表名称等于模型名称,而无需进行任何修改：

```js
sequelize.define('User', {
  // ... (属性)
}, {
  freezeTableName: true
});
```

上面的示例将创建一个名为 `User` 的模型,该模型指向一个也名为 `User` 的表.

也可以为 sequelize 实例全局定义此行为：

```js
const sequelize = new Sequelize('sqlite::memory:', {
  define: {
    freezeTableName: true
  }
});
```

这样,所有表将使用与模型名称相同的名称.

### 直接提供表名

你也可以直接直接告诉 Sequelize 表名称：

```js
sequelize.define('User', {
  // ... (属性)
}, {
  tableName: 'Employees'
});
```

## 模型同步

定义模型时,你要告诉 Sequelize 有关数据库中表的一些信息. 但是,如果该表实际上不存在于数据库中怎么办？ 如果存在,但具有不同的列,较少的列或任何其他差异,该怎么办？

这就是模型同步的来源.可以通过调用一个异步函数(返回一个Promise)[`model.sync(options)`](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-sync). 通过此调用,Sequelize 将自动对数据库执行 SQL 查询. 请注意,这仅更改数据库中的表,而不更改 JavaScript 端的模型.

* `User.sync()` - 如果表不存在,则创建该表(如果已经存在,则不执行任何操作)
* `User.sync({ force: true })` - 将创建表,如果表已经存在,则将其首先删除
* `User.sync({ alter: true })` - 这将检查数据库中表的当前状态(它具有哪些列,它们的数据类型等),然后在表中进行必要的更改以使其与模型匹配.

示例:

```js
await User.sync({ force: true });
console.log("用户模型表刚刚(重新)创建！");
```

### 一次同步所有模型

你可以使用 [`sequelize.sync()`](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-method-sync) 自动同步所有模型. 示例：

```js
await sequelize.sync({ force: true });
console.log("所有模型均已成功同步.");
```

### 删除表

删除与模型相关的表：

```js
await User.drop();
console.log("用户表已删除!");
```

删除所有表：

```js
await sequelize.drop();
console.log("所有表已删除!");
```

### 数据库安全检查

如上所示,`sync`和`drop`操作是破坏性的. Sequelize 使用 `match` 参数作为附加的安全检查,该检查将接受 RegExp：

```js
// 仅当数据库名称以 '_test' 结尾时,它才会运行.sync()
sequelize.sync({ force: true, match: /_test$/ });
```

### 生产环境同步

如上所示,`sync({ force: true })` 和 `sync({ alter: true })` 可能是破坏性操作. 因此,不建议将它们用于生产级软件中. 相反,应该在 [Sequelize CLI](https://github.com/sequelize/cli) 的帮助下使用高级概念 [Migrations](migrations.html)(迁移) 进行同步.

## 时间戳

默认情况下,Sequelize 使用数据类型 `DataTypes.DATE` 自动向每个模型添加 `createdAt` 和 `updatedAt` 字段. 这些字段会自动进行管理 - 每当你使用Sequelize 创建或更新内容时,这些字段都会被自动设置. `createdAt` 字段将包含代表创建时刻的时间戳,而 `updatedAt` 字段将包含最新更新的时间戳.

**注意：** 这是在 Sequelize 级别完成的(即未使用 *SQL触发器* 完成). 这意味着直接 SQL 查询(例如,通过任何其他方式在不使用 Sequelize 的情况下执行的查询)将不会导致这些字段自动更新.

对于带有 `timestamps: false` 参数的模型,可以禁用此行为：

```js
sequelize.define('User', {
  // ... (属性)
}, {
  timestamps: false
});
```

也可以只启用 `createdAt`/`updatedAt` 之一,并为这些列提供自定义名称：

```js
class Foo extends Model {}
Foo.init({ /* 属性 */ }, {
  sequelize,

  // 不要忘记启用时间戳！
  timestamps: true,

  // 不想要 createdAt
  createdAt: false,

  // 想要 updatedAt 但是希望名称叫做 updateTimestamp
  updatedAt: 'updateTimestamp'
});
```

## 列声明简写语法

如果关于列的唯一指定内容是其数据类型,则可以缩短语法：

```js
// 例如:
sequelize.define('User', {
  name: {
    type: DataTypes.STRING
  }
});

// 可以简写为:
sequelize.define('User', { name: DataTypes.STRING });
```

## 默认值

默认情况下,Sequelize 假定列的默认值为 `NULL`. 可以通过将特定的 `defaultValue` 传递给列定义来更改此行为：

```js
sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    defaultValue: "John Doe"
  }
});
```

一些特殊的值,例如 `Sequelize.NOW`,也能被接受：

```js
sequelize.define('Foo', {
  bar: {
    type: DataTypes.DATETIME,
    defaultValue: Sequelize.NOW
    // 这样,当前日期/时间将用于填充此列(在插入时)
  }
});
```

## 数据类型

你在模型中定义的每一列都必须具有数据类型. Sequelize 提供[很多内置数据类型](https://github.com/sequelize/sequelize/blob/master/lib/data-types.js). 要访问内置数据类型,必须导入 `DataTypes`：

```js
const { DataTypes } = require("sequelize"); // 导入内置数据类型
```

### 字符串

```js
DataTypes.STRING             // VARCHAR(255)
DataTypes.STRING(1234)       // VARCHAR(1234)
DataTypes.STRING.BINARY      // VARCHAR BINARY
DataTypes.TEXT               // TEXT
DataTypes.TEXT('tiny')       // TINYTEXT
DataTypes.CITEXT             // CITEXT          仅 PostgreSQL 和 SQLite.
```

### 布尔

```js
DataTypes.BOOLEAN            // TINYINT(1)
```

### 数字

```js
DataTypes.INTEGER            // INTEGER
DataTypes.BIGINT             // BIGINT
DataTypes.BIGINT(11)         // BIGINT(11)

DataTypes.FLOAT              // FLOAT
DataTypes.FLOAT(11)          // FLOAT(11)
DataTypes.FLOAT(11, 10)      // FLOAT(11,10)

DataTypes.REAL               // REAL            仅 PostgreSQL.
DataTypes.REAL(11)           // REAL(11)        仅 PostgreSQL.
DataTypes.REAL(11, 12)       // REAL(11,12)     仅 PostgreSQL.

DataTypes.DOUBLE             // DOUBLE
DataTypes.DOUBLE(11)         // DOUBLE(11)
DataTypes.DOUBLE(11, 10)     // DOUBLE(11,10)

DataTypes.DECIMAL            // DECIMAL
DataTypes.DECIMAL(10, 2)     // DECIMAL(10,2)
```

#### 无符号和零填充整数 - 仅限于MySQL/MariaDB

在 MySQL 和 MariaDB 中,可以将数据类型`INTEGER`, `BIGINT`, `FLOAT` 和 `DOUBLE` 设置为无符号或零填充(或两者),如下所示：

```js
DataTypes.INTEGER.UNSIGNED
DataTypes.INTEGER.ZEROFILL
DataTypes.INTEGER.UNSIGNED.ZEROFILL
// 你还可以指定大小,即INTEGER(10)而不是简单的INTEGER
// 同样适用于 BIGINT, FLOAT 和 DOUBLE
```

### 日期

```js
DataTypes.DATE       // DATETIME 适用于 mysql / sqlite, 带时区的TIMESTAMP 适用于 postgres
DataTypes.DATE(6)    // DATETIME(6) 适用于 mysql 5.6.4+. 支持6位精度的小数秒
DataTypes.DATEONLY   // 不带时间的 DATE
```

### UUID

对于 UUID,使用 `DataTypes.UUID`. 对于 PostgreSQL 和 SQLite,它会是 `UUID` 数据类型;对于 MySQL,它则变成`CHAR(36)`. Sequelize 可以自动为这些字段生成 UUID,只需使用 `Sequelize.UUIDV1` 或 `Sequelize.UUIDV4` 作为默认值即可：

```js
{
  type: DataTypes.UUID,
  defaultValue: Sequelize.UUIDV4 // 或 Sequelize.UUIDV1
}
```

### 其它

还有其他数据类型,请参见[其它数据类型](../other-topics/other-data-types.md).

## 列参数

在定义列时,除了指定列的 `type` 以及上面提到的 `allowNull` 和 `defaultValue` 参数外,还有很多可用的参数. 下面是一些示例.

```js
const { Model, DataTypes, Deferrable } = require("sequelize");

class Foo extends Model {}
Foo.init({
  // 实例化将自动将 flag 设置为 true (如果未设置)
  flag: { type: DataTypes.BOOLEAN, allowNull: false, defaultValue: true },

  // 日期的默认值 => 当前时间
  myDate: { type: DataTypes.DATE, defaultValue: DataTypes.NOW },

  // 将 allowNull 设置为 false 将为该列添加 NOT NULL,
  // 这意味着如果该列为 null,则在执行查询时将从数据库引发错误.
  // 如果要在查询数据库之前检查值是否不为 null,请查看下面的验证部分.
  title: { type: DataTypes.STRING, allowNull: false },

  // 创建两个具有相同值的对象将引发错误.
  // unique 属性可以是布尔值或字符串.
  // 如果为多个列提供相同的字符串,则它们将形成一个复合唯一键.
  uniqueOne: { type: DataTypes.STRING,  unique: 'compositeIndex' },
  uniqueTwo: { type: DataTypes.INTEGER, unique: 'compositeIndex' },

  // unique 属性是创建唯一约束的简写.
  someUnique: { type: DataTypes.STRING, unique: true },

  // 继续阅读有关主键的更多信息
  identifier: { type: DataTypes.STRING, primaryKey: true },

  // autoIncrement 可用于创建 auto_incrementing 整数列
  incrementMe: { type: DataTypes.INTEGER, autoIncrement: true },

  // 你可以通过 'field' 属性指定自定义列名称：
  fieldWithUnderscores: { type: DataTypes.STRING, field: 'field_with_underscores' },

  // 可以创建外键：
  bar_id: {
    type: DataTypes.INTEGER,

    references: {
      // 这是对另一个模型的参考
      model: Bar,

      // 这是引用模型的列名
      key: 'id',

      // 使用 PostgreSQL,可以通过 Deferrable 类型声明何时检查外键约束.
      deferrable: Deferrable.INITIALLY_IMMEDIATE
      // 参数:
      // - `Deferrable.INITIALLY_IMMEDIATE` - 立即检查外键约束
      // - `Deferrable.INITIALLY_DEFERRED` - 将所有外键约束检查推迟到事务结束
      // - `Deferrable.NOT` - 完全不推迟检查(默认) - 这将不允许你动态更改事务中的规则
    }
  },

  // 注释只能添加到 MySQL,MariaDB,PostgreSQL 和 MSSQL 的列中
  commentMe: {
    type: DataTypes.INTEGER,
    comment: '这是带有注释的列'
  }
}, {
  sequelize,
  modelName: 'foo',

  // 在上面的属性中使用 `unique: true` 与在模型的参数中创建索引完全相同：
  indexes: [{ unique: true, fields: ['someUnique'] }]
});
```

## 利用模型作为类

Sequelize 模型是 [ES6 类](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes). 你可以非常轻松地添加自定义实例或类级别的方法.

```js
class User extends Model {
  static classLevelMethod() {
    return 'foo';
  }
  instanceLevelMethod() {
    return 'bar';
  }
  getFullname() {
    return [this.firstname, this.lastname].join(' ');
  }
}
User.init({
  firstname: Sequelize.TEXT,
  lastname: Sequelize.TEXT
}, { sequelize });

console.log(User.classLevelMethod()); // 'foo'
const user = User.build({ firstname: 'Jane', lastname: 'Doe' });
console.log(user.instanceLevelMethod()); // 'bar'
console.log(user.getFullname()); // 'Jane Doe'
```