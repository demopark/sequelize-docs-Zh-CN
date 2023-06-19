# Extending Data Types - 扩展数据类型

你尝试实现的类型很可能已经包含在[数据类型](../core-concepts/model-basics.md)中. 如果不包括新的数据类型, 本手册将展示如何自己编写或扩展现有数据类型.

## 创建一个新的数据类型

DataType 是一个扩展于 `DataTypes.ABSTRACT` 并实现其 `toSql` 方法的类:

```typescript
import { Sequelize, DataTypes } from '@sequelize/core';

// 所有数据类型都必须继承自 DataTypes.ABSTRACT.
export class MyDateType extends DataTypes.ABSTRACT {
  // toSql 必须返回将在 CREATE TABLE 语句中使用的 SQL.
  toSql() {
    return 'TIMESTAMP';
  }
}
```

然后, 你可以在模型中使用新的数据类型:

```typescript
import { MyDateType } from './custom-types.js';

const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  birthday: {
    // 新数据类型
    type: MyDateType,
  },
}, { timestamps: false, noPrimaryKey: true, underscored: true });

await User.sync();
```

以上将产生以下SQL:

```sql
CREATE TABLE IF NOT EXISTS "users" (
  "birthday" TIMESTAMP
);
```

### 验证用户输入

现在, 我们的数据类型非常简单. 它不进行任何规范化, 而是按原样将值传递给数据库. 它具有与我们将 [属性的类型设置为字符串](../other-topics/other-data-types.md) 相同的基本行为.

你可以实现一系列方法来更改数据类型的行为:

- `validate(value): void` - 在模型实例上设置值时调用此方法. 如果它返回 `false`, 该值将被拒绝.
- `sanitize(value): unknown` - 在模型实例上设置值时调用此方法. 它在验证之前被调用. 你可以使用它来规范化一个值, 例如将字符串转换为 Date 对象.
- `areValuesEqual(a, b): boolean` - 在比较数据类型的两个值时, 在确定需要保存模型的哪些属性时调用此方法. 
   如果它返回 `false`, 新值将被保存. 默认情况下, 它使用 lodash 的 isEqual 方法.

```typescript
import { Sequelize, DataTypes, ValidationErrorItem } from '@sequelize/core';

export class MyDateType extends DataTypes.ABSTRACT<Date> {
  toSql() {
    return 'TIMESTAMP';
  }
  
  sanitize(value: unknown): unknown {
    if (value instanceof Date) {
      return value;
    }
    
    if (typeof value === 'string') {
      return new Date(value);
    }
    
    throw new ValidationErrorItem('Invalid date');
  }
  
  validate(value: unknown): void {
    if (!(value instanceof Date)) {
      ValidationErrorItem.throwDataTypeValidationError('Value must be a Date object');
    }
    
    if (Number.isNaN(value.getTime())) {
      ValidationErrorItem.throwDataTypeValidationError('Value is an Invalid Date');
    }
  }
  
  sanitize(value: unknown): unknown {
    if (typeof value === 'string') {
      return new Date(value);
    }
  }
}
```

### 序列化 & 反序列化

我们有 4 个方法可以用来定义数据类型在与数据库交互时如何序列化和反序列化值:

- `parseDatabaseValue(value): unknown`: 转换从数据库中检索的值.
- `toBindableValue(value): unknown`: 使用 [绑定参数](../core-concepts/raw-queries.md) 时将值转换为连接器库接受的值.
- `escape(value): string`: 转义用于内联原始 SQL 的值, 例如在使用 [替换](../core-concepts/raw-queries.md) 时. 默认情况下, 如果 `toBindableValue` 返回一个字符串, 此方法将将该字符串转义为 SQL 字符串.

```typescript
import { DataTypes, StringifyOptions } from '@sequelize/core';

export class MyDateType extends DataTypes.ABSTRACT<Date> {
  // [...] 截断的例子
  
  parseDatabaseValue(value: unknown): Date {
    assert(typeof value === 'string', 'Expected to receive a string from the database');
    
    return new Date(value);
  }

  toBindableValue(value: Date): unknown {
    return value.toISOString();
  }
  
  escape(value: Date, options: StringifyOptions): string {
    return options.dialect.escapeString(value.toISOString());
  }
}
```

## 修改现有数据类型

你可以继承现有数据类型的实现来自定义其行为. 比如, 让你的类扩展你希望修改的数据类型, 而不是由 `DataTypes.ABSTRACT` 替代. 

请注意以下示例是如何继承自 `DataTypes.STRING` 而不是 `DataTypes.ABSTRACT` 的:

```typescript
import { Sequelize, DataTypes } from '@sequelize/core';

export class MyStringType extends DataTypes.STRING {
  toSql() {
    return 'TEXT';
  }
}
```

就像自定义数据类型一样, 使用你的数据类型类而不是你正在扩展的类型:

```typescript
import { MyStringType } from './custom-types.js';

const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  firstName: {
    // 新数据类型
    type: MyStringType,
  },
}, { timestamps: false, noPrimaryKey: true, underscored: true });

await User.sync();
```

## 限制

某些方言支持通过使用 [`CREATE TYPE`](https://www.postgresql.org/docs/current/sql-createtype.html) 语句创建自定义 SQL 数据类型. postgres 中的枚举就是这种情况.

使用 `DataTypes.ENUM` 时, Sequelize 会自动在数据库中创建枚举类型. 这对于自定义类型是不可能的.
如果你需要创建自定义类型, 则需要先在数据库中手动创建它, 然后才能在你的某个模型中使用它.
