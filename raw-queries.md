# Raw queries - 原始查询

由于常常使用简单的方式来执行原始/已经准备好的SQL查询，所以可以使用 `sequelize.query` 函数。

默认情况下，函数将返回两个参数 - 一个结果数组，以及一个包含元数据（受影响的行等）的对象。 请注意，由于这是一个原始查询，所以元数据（属性名称等）是具体的方言。 某些方言返回元数据 "within" 结果对象（作为数组上的属性）。 但是，将永远返回两个参数，但对于MSSQL和MySQL，它将是对同一对象的两个引用。

```js
sequelize.query("UPDATE users SET y = 42 WHERE x = 12").spread((results, metadata) => {
  // 结果将是一个空数组，元数据将包含受影响的行数。
})
```

在不需要访问元数据的情况下，您可以传递一个查询类型来告诉后续如何格式化结果。 例如，对于一个简单的选择查询你可以做：

```js
sequelize.query("SELECT * FROM `users`", { type: sequelize.QueryTypes.SELECT})
  .then(users => {
    // 我们不需要在这里延伸，因为只有结果将返回给选择查询
  })
```

还有其他几种查询类型可用。 [详细了解来源](https://github.com/sequelize/sequelize/blob/master/lib/query-types.js)

第二种选择是模型。 如果传递模型，返回的数据将是该模型的实例。

```js
// Callee 是模型定义。 这样您就可以轻松地将查询映射到预定义的模型
sequelize.query('SELECT * FROM projects', { model: Projects }).then(projects => {
  // 每个记录现在将是Project的一个实例
})
```

## 替换

查询中的替换可以通过两种不同的方式完成：使用命名参数（以`：`开头），或者由`？`表示的未命名参数。 替换在options对象中传递。

* 如果传递一个数组, `?` 将按照它们在数组中出现的顺序被替换
* 如果传递一个对象, `:key` 将替换为该对象的键。 如果对象包含在查询中找不到的键，则会抛出异常，反之亦然。

```js
sequelize.query('SELECT * FROM projects WHERE status = ?',
  { replacements: ['active'], type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT * FROM projects WHERE status = :status ',
  { replacements: { status: 'active' }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```

数组替换将自动处理，以下查询将搜索状态与值数组匹配的项目。

```js
sequelize.query('SELECT * FROM projects WHERE status IN(:status) ',
  { replacements: { status: ['active', 'inactive'] }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```

要使用通配符运算符 ％，请将其附加到你的替换中。 以下查询与名称以“ben”开头的用户相匹配。

```js
sequelize.query('SELECT * FROM users WHERE name LIKE :search_name ',
  { replacements: { search_name: 'ben%'  }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```

## 绑定参数

绑定参数就像替换。 除非替换被转义并在查询发送到数据库之前通过后续插入到查询中，而将绑定参数发送到SQL查询文本之外的数据库。 查询可以具有绑定参数或替换。

只有SQLite和PostgreSQL支持绑定参数。 其他方言会将它们插入到SQL查询中，就像替换一样。 绑定参数由 `$1, $2, ... (numeric)` 或 `$key (alpha-numeric)` 引用。这是独立于方言。

* 如果传递一个数组, `$1` 被绑定到数组中的第一个元素 (`bind[0]`)。
* 如果传递一个对象, `$key` 绑定到 `object['key']`。 每个键必须以非数字字符开始。 `$1` 不是一个有效的键，即使 `object['1']` 存在。
* 在这两种情况下 `$$` 可以用来转义一个 `$` 字符符号.

数组或对象必须包含所有绑定的值，或者Sequelize将抛出异常。 这甚至适用于数据库可能忽略绑定参数的情况。

数据库可能会增加进一步的限制。 绑定参数不能是SQL关键字，也不能是表或列名。 引用的文本或数据也忽略它们。 在PostgreSQL中，如果不能从上下文 `$1::varchar` 推断类型，那么也可能需要对其进行类型转换。

```js
sequelize.query('SELECT *, "text with literal $$1 and literal $$status" as t FROM projects WHERE status = $1',
  { bind: ['active'], type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})

sequelize.query('SELECT *, "text with literal $$1 and literal $$status" as t FROM projects WHERE status = $status',
  { bind: { status: 'active' }, type: sequelize.QueryTypes.SELECT }
).then(projects => {
  console.log(projects)
})
```

