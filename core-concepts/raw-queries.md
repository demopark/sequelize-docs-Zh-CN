# Raw Queries - 原始查询

由于常常使用简单的方式来执行原始/已经准备好的SQL查询,因此可以使用 [`sequelize.query`](/api/v7/classes/Sequelize.html#query) 方法.

默认情况下,函数将返回两个参数 - 一个结果数组,以及一个包含元数据(例如受影响的行数等)的对象. 请注意,由于这是一个原始查询,所以元数据都是具体的方言. 某些方言返回元数据 "within" 结果对象(作为数组上的属性). 但是,将永远返回两个参数,但对于MSSQL和MySQL,它将是对同一对象的两个引用.

```js
const [results, metadata] = await sequelize.query("UPDATE users SET y = 42 WHERE x = 12");
// 结果将是一个空数组,元数据将包含受影响的行数.
```

在不需要访问元数据的情况下,你可以传递一个查询类型来告诉后续如何格式化结果. 例如,对于一个简单的选择查询你可以做:

```js
const { QueryTypes } = require('@sequelize/core');
const users = await sequelize.query("SELECT * FROM `users`", { type: QueryTypes.SELECT });
// 我们不需要在这里分解结果 - 结果会直接返回

```

还有其他几种查询类型可用. [详细了解来源](https://github.com/sequelize/sequelize/blob/main/packages/core/src/query-types.ts).

第二种选择是模型. 如果传递模型,返回的数据将是该模型的实例.

```js
// Callee 是模型定义. 这样你就可以轻松地将查询映射到预定义的模型
const projects = await sequelize.query('SELECT * FROM projects', {
  model: Projects,
  mapToModel: true // 如果你有任何映射字段,则在此处传递 true
});
// 现在,`projects` 的每个元素都是 Project 的一个实例
```

查看 [Query API 参考](/api/v7/classes/Sequelize.html#query)中的更多参数. 以下是一些例子:

```js
const { QueryTypes } = require('@sequelize/core');
await sequelize.query('SELECT 1', {
  // 用于记录查询的函数(或false)
  // 将调用发送到服务器的每个SQL查询.
  logging: console.log,

  // 如果plain为true,则sequelize将仅返回结果集的第一条记录. 
  // 如果是false,它将返回所有记录.
  plain: false,

  // 如果你没有查询的模型定义,请将此项设置为true.
  raw: false,

  // 你正在执行的查询类型. 查询类型会影响结果在传回之前的格式.
  type: QueryTypes.SELECT
});

// 注意第二个参数为null！
// 即使我们在这里声明了一个被调用对象,
// raw: true 也会取代并返回一个原始对象.

console.log(await sequelize.query('SELECT * FROM projects', { raw: true }));
```

## "Dotted" 属性 和 `nest` 参数

如果表的属性名称包含点,则可以通过设置 `nest: true` 参数将生成的对象变为嵌套对象. 这可以通过 [dottie.js](https://github.com/mickhansen/dottie.js/) 在后台实现. 见下文：

* 不使用 `nest: true`:

  ```js
  const { QueryTypes } = require('@sequelize/core');
  const records = await sequelize.query('select 1 as `foo.bar.baz`', {
    type: QueryTypes.SELECT
  });
  console.log(JSON.stringify(records[0], null, 2));
  ```

  ```json
  {
    "foo.bar.baz": 1
  }
  ```

* 使用 `nest: true`:

  ```js
  const { QueryTypes } = require('@sequelize/core');
  const records = await sequelize.query('select 1 as `foo.bar.baz`', {
    nest: true,
    type: QueryTypes.SELECT
  });
  console.log(JSON.stringify(records[0], null, 2));
  ```

  ```json
  {
    "foo": {
      "bar": {
        "baz": 1
      }
    }
  }
  ```

## 替换

替换是在查询中传递变量的一种方式. 它们是 [绑定参数](#绑定参数) 的替代方法.

替换参数和绑定参数之间的区别在于, 替换参数在查询发送到数据库之前被转义并由 Sequelize 插入到查询中, 
而绑定参数与 SQL 查询文本分开发送到数据库, 并由数据库本身进行 "转义".

替换可以用两种不同的方式编写:

- 使用数字标识符 (由一个 `?` 表示). `replacements` 参数必须是一个数组. 这些值将按照它们在数组和查询中出现的顺序被替换.
- 或者使用字母标识符 (例如 `:firstName`, `:status`, 等…). 这些标识符遵循通用标识符规则 (仅限字母数字和下划线, 不能以数字开头). `replacements` 参数必须是包含每个参数(没有 `:` 前缀)的普通对象.

`replacements` 参数必须包含所有绑定值, 否则 Sequelize 将抛出错误.

示例:

```js
const { QueryTypes } = require('@sequelize/core');

await sequelize.query(
  'SELECT * FROM projects WHERE status = ?',
  {
    replacements: ['active'],
    type: QueryTypes.SELECT
  }
);

await sequelize.query(
  'SELECT * FROM projects WHERE status = :status',
  {
    replacements: { status: 'active' },
    type: QueryTypes.SELECT,
  },
);
```

当使用 `LIKE` 等运算符时, 请记住替换中的特殊字符确实保留其特殊含义.

例如 以下查询匹配名称以 "ben" 开头的用户:

```js
const { QueryTypes } = require('@sequelize/core');

await sequelize.query(
  'SELECT * FROM users WHERE name LIKE :searchName',
  {
    replacements: { searchName: 'ben%' },
    type: QueryTypes.SELECT
  }
);
```

数组替换将自动处理,以下查询将搜索状态与值数组匹配的项目.

```js
const { QueryTypes } = require('sequelize');

await sequelize.query(
  'SELECT * FROM projects WHERE status IN(:status)',
  {
    replacements: { status: ['active', 'inactive'] },
    type: QueryTypes.SELECT
  }
);
```

要使用通配符运算符 `％`,请将其附加到你的替换中. 以下查询与名称以 'ben' 开头的用户相匹配.

```js
const { QueryTypes } = require('sequelize');

await sequelize.query(
  'SELECT * FROM users WHERE name LIKE :search_name',
  {
    replacements: { search_name: 'ben%' },
    type: QueryTypes.SELECT
  }
);
```

**警告**

Sequelize 目前不支持[指定替换的数据类型](https://github.com/sequelize/sequelize/issues/14410) 的方法, 并且会在序列化之前尝试猜测其类型.  

例如, 数组不会被序列化为 SQL 的 "ARRAY" 类型. 取而代之的是以下查询：

```js
const { QueryTypes } = require('@sequelize/core');

await sequelize.query(
  'SELECT * FROM projects WHERE status IN (:status)',
  {
    replacements: { status: ['active', 'inactive'] },
    type: QueryTypes.SELECT
  }
);
```

将导产生此 SQL:

```sql
SELECT * FROM projects WHERE status IN ('active', 'inactive')
```

在实现此类功能之前, 你可以使用 [绑定参数](#绑定参数) 并将其强制转换.

## 绑定参数

绑定参数是一种在查询中传递变量的方法. 它们是 [替换](#替换) 的替代品.

替换参数和绑定参数之间的区别在于, 替换参数在查询发送到数据库之前被转义并由 Sequelize 插入到查询中, 
而绑定参数与 SQL 查询文本分开发送到数据库, 并由数据库本身 "转义".

一个查询可以同时具有绑定参数和替换参数.

每个数据库对绑定参数使用不同的语法, 但 Sequelize 提供了自己统一的层.

与你使用哪个数据库无关紧要, 在 Sequelize 中, 绑定参数是按照类似 postgres 的语法编写的. 你也可以:

- 使用数字标识符 (例如 `$1`, `$2`, 等...). 请注意, 这些标识符从 1 开始, 而不是 0. `bind` 参数必须是一个数组, 其中包含查询中使用的每个标识符的值(`$1` 绑定到数组中的第一个元素(`bind[0]`), 以此类推...).
- 使用字母标识符 (例如 `$firstName`, `$status`, 等...). 这些标识符遵循通用标识符规则 (仅限字母数字和下划线, 不能以数字开头). `bind` 参数必须是一个普通对象, 其中包含每个绑定参数(没有 `$` 前缀).

`bind` 餐食必须包含所有绑定值, 否则 Sequelize 将抛出错误.

**注意**

绑定参数只能用于数据值. 绑定参数不能用于动态更改表名, 列名或查询的其他非数据值部分.

你的数据库可能对绑定参数有进一步的限制.

示例:

```js
const { QueryTypes } = require('@sequelize/core');

await sequelize.query(
  'SELECT * FROM projects WHERE status = $1',
  {
    bind: ['active'],
    type: QueryTypes.SELECT,
  },
);

await sequelize.query(
  'SELECT * FROM projects WHERE status = $status',
  {
    bind: { status: 'active' },
    type: QueryTypes.SELECT,
  },
);
```

Sequelize 目前不支持[指定绑定参数的数据类型](https://github.com/sequelize/sequelize/issues/14410).
在实现此类功能之前, 如果需要更改绑定参数的数据类型, 则可以强制转换它们:

```js
const { QueryTypes } = require('@sequelize/core');

await sequelize.query(
  'SELECT * FROM projects WHERE id = CAST($1 AS int)',
  {
    bind: [5],
    type: QueryTypes.SELECT,
  },
);
```

**注意**

某些方言 (例如 PostgreSQL 和 IBM Db2) 支持更简洁的强制转换语法, 你可以根据需要使用该语法:

```typescript
await sequelize.query('SELECT * FROM projects WHERE id = $1::int');
```
