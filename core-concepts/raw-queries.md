# Raw Queries - 原始查询

由于常常使用简单的方式来执行原始/已经准备好的SQL查询,因此可以使用 `sequelize.query` 方法.

默认情况下,函数将返回两个参数 - 一个结果数组,以及一个包含元数据(例如受影响的行数等)的对象. 请注意,由于这是一个原始查询,所以元数据都是具体的方言. 某些方言返回元数据 "within" 结果对象(作为数组上的属性). 但是,将永远返回两个参数,但对于MSSQL和MySQL,它将是对同一对象的两个引用.

```js
const [results, metadata] = await sequelize.query("UPDATE users SET y = 42 WHERE x = 12");
// 结果将是一个空数组,元数据将包含受影响的行数.
```

在不需要访问元数据的情况下,你可以传递一个查询类型来告诉后续如何格式化结果. 例如,对于一个简单的选择查询你可以做:

```js
const { QueryTypes } = require('sequelize');
const users = await sequelize.query("SELECT * FROM `users`", { type: QueryTypes.SELECT });
// 我们不需要在这里分解结果 - 结果会直接返回

```

还有其他几种查询类型可用. [详细了解来源](https://github.com/sequelize/sequelize/blob/main/src/query-types.ts).

第二种选择是模型. 如果传递模型,返回的数据将是该模型的实例.

```js
// Callee 是模型定义. 这样你就可以轻松地将查询映射到预定义的模型
const projects = await sequelize.query('SELECT * FROM projects', {
  model: Projects,
  mapToModel: true // 如果你有任何映射字段,则在此处传递 true
});
// 现在,`projects` 的每个元素都是 Project 的一个实例
```

查看 [Query API 参考](class/lib/sequelize.js~Sequelize.html#instance-method-query)中的更多参数. 以下是一些例子:

```js
const { QueryTypes } = require('sequelize');
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
  const { QueryTypes } = require('sequelize');
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
  const { QueryTypes } = require('sequelize');
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

查询中的替换可以通过两种不同的方式完成:使用命名参数(以`:`开头),或者由`？`表示的未命名参数. 替换在options对象中传递.

* 如果传递一个数组, `?` 将按照它们在数组中出现的顺序被替换
* 如果传递一个对象, `:key` 将替换为该对象的键. 如果对象包含在查询中找不到的键,则会抛出异常,反之亦然.

```js
const { QueryTypes } = require('sequelize');

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

## 绑定参数

绑定参数就像替换. 除非替换被转义并在查询发送到数据库之前通过后续插入到查询中,而将绑定参数发送到SQL查询文本之外的数据库. 查询可以具有绑定参数或替换.绑定参数由 `$1, $2, ... (numeric)` 或 `$key (alpha-numeric)` 引用.这是独立于方言的.

* 如果传递一个数组, `$1` 被绑定到数组中的第一个元素 (`bind[0]`).
* 如果传递一个对象, `$key` 绑定到 `object['key']`. 每个键必须以非数字字符开始. `$1` 不是一个有效的键,即使 `object['1']` 存在.
* 在这两种情况下 `$$` 可以用来转义一个 `$` 字符符号.

数组或对象必须包含所有绑定的值,或者Sequelize将抛出异常. 这甚至适用于数据库可能忽略绑定参数的情况.

数据库可能会增加进一步的限制. 绑定参数不能是SQL关键字,也不能是表或列名. 引用的文本或数据也忽略它们. 在PostgreSQL中,如果不能从上下文 `$1::varchar` 推断类型,那么也可能需要对其进行类型转换.

```js
const { QueryTypes } = require('sequelize');

await sequelize.query(
  'SELECT *, "text with literal $$1 and literal $$status" as t FROM projects WHERE status = $1',
  {
    bind: ['active'],
    type: QueryTypes.SELECT
  }
);

await sequelize.query(
  'SELECT *, "text with literal $$1 and literal $$status" as t FROM projects WHERE status = $status',
  {
    bind: { status: 'active' },
    type: QueryTypes.SELECT
  }
);
```

