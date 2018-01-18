# Querying - 查询

## 属性

想要只选择某些属性，可以使用 `attributes` 选项。 通常是传递一个数组：

```js
Model.findAll({
  attributes: ['foo', 'bar']
});
```
```sql
SELECT foo, bar ...
```

属性可以使用嵌套数组来重命名：

```js
Model.findAll({
  attributes: ['foo', ['bar', 'baz']]
});
```
```sql
SELECT foo, bar AS baz ...
```

也可以使用 `sequelize.fn` 来进行聚合：

```js
Model.findAll({
  attributes: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});
```
```sql
SELECT COUNT(hats) AS no_hats ...
```

使用聚合功能时，必须给它一个别名，以便能够从模型中访问它。 在上面的例子中，您可以使用  `instance.get('no_hats')` 获得帽子数量。

有时，如果您只想添加聚合，则列出模型的所有属性可能令人厌烦：

```js
// This is a tiresome way of getting the number of hats...
Model.findAll({
  attributes: ['id', 'foo', 'bar', 'baz', 'quz', [sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]
});

// This is shorter, and less error prone because it still works if you add / remove attributes
Model.findAll({
  attributes: { include: [[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']] }
});
```
```sql
SELECT id, foo, bar, baz, quz, COUNT(hats) AS no_hats ...
```

同样，它也可以删除一些指定的属性：

```js
Model.findAll({
  attributes: { exclude: ['baz'] }
});
```
```sql
SELECT id, foo, bar, quz ...
```


## Where

无论您是通过 findAll/find 或批量 updates/destroys 进行查询，都可以传递一个 `where` 对象来过滤查询。

`where` 通常用 attribute:value 键值对获取一个对象，其中 value 可以是匹配等式的数据或其他运算符的键值对象。

也可以通过嵌套 `or` 和 `and` `运算符` 的集合来生成复杂的 AND/OR 条件。

### 基础
```js
const Op = Sequelize.Op;

Post.findAll({
  where: {
    authorId: 2
  }
});
// SELECT * FROM post WHERE authorId = 2

Post.findAll({
  where: {
    authorId: 12,
    status: 'active'
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';

Post.findAll({
  where: {
    [Op.or]: [{authorId: 12}, {authorId: 13}]
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.findAll({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;

Post.destroy({
  where: {
    status: 'inactive'
  }
});
// DELETE FROM post WHERE status = 'inactive';

Post.update({
  updatedAt: null,
}, {
  where: {
    deletedAt: {
      [Op.ne]: null
    }
  }
});
// UPDATE post SET updatedAt = null WHERE deletedAt NOT NULL;

Post.findAll({
  where: sequelize.where(sequelize.fn('char_length', sequelize.col('status')), 6)
});
// SELECT * FROM post WHERE char_length(status) = 6;
```

### 操作符

Sequelize 可用于创建更复杂比较的符号运算符 -

```js
const Op = Sequelize.Op

[Op.and]: {a: 5}           // 且 (a = 5)
[Op.or]: [{a: 5}, {a: 6}]  // (a = 5 或 a = 6)
[Op.gt]: 6,                // id > 6
[Op.gte]: 6,               // id >= 6
[Op.lt]: 10,               // id < 10
[Op.lte]: 10,              // id <= 10
[Op.ne]: 20,               // id != 20
[Op.eq]: 3,                // = 3
[Op.not]: true,            // 不是 TRUE
[Op.between]: [6, 10],     // 在 6 和 10 之间
[Op.notBetween]: [11, 15], // 不在 11 和 15 之间
[Op.in]: [1, 2],           // 在 [1, 2] 之中
[Op.notIn]: [1, 2],        // 不在 [1, 2] 之中
[Op.like]: '%hat',         // 包含 '%hat'
[Op.notLike]: '%hat'       // 不包含 '%hat'
[Op.iLike]: '%hat'         // 包含 '%hat' (不区分大小写)  (仅限 PG)
[Op.notILike]: '%hat'      // 不包含 '%hat'  (仅限 PG)
[Op.regexp]: '^[h|a|t]'    // 匹配正则表达式/~ '^[h|a|t]' (仅限 MySQL/PG)
[Op.notRegexp]: '^[h|a|t]' // 不匹配正则表达式/!~ '^[h|a|t]' (仅限 MySQL/PG)
[Op.iRegexp]: '^[h|a|t]'    // ~* '^[h|a|t]' (仅限 PG)
[Op.notIRegexp]: '^[h|a|t]' // !~* '^[h|a|t]' (仅限 PG)
[Op.like]: { [Op.any]: ['cat', 'hat']} // 包含任何数组['cat', 'hat'] - 同样适用于 iLike 和 notLike
[Op.overlap]: [1, 2]       // && [1, 2] (PG数组重叠运算符)
[Op.contains]: [1, 2]      // @> [1, 2] (PG数组包含运算符)
[Op.contained]: [1, 2]     // <@ [1, 2] (PG数组包含于运算符)
[Op.any]: [2,3]            // 任何数组[2, 3]::INTEGER (仅限PG)

[Op.col]: 'user.organization_id' // = 'user'.'organization_id', 使用数据库语言特定的列标识符, 本例使用 PG
```

#### 范围选项

所有操作符都支持支持的范围类型查询。

请记住，提供的范围值也可以[定义绑定的 inclusion/exclusion](/manual/tutorial/models-definition.html#range-types)。

```js
// 所有上述相等和不相等的操作符加上以下内容:

[Op.contains]: 2           // @> '2'::integer (PG range contains element operator)
[Op.contains]: [1, 2]      // @> [1, 2) (PG range contains range operator)
[Op.contained]: [1, 2]     // <@ [1, 2) (PG range is contained by operator)
[Op.overlap]: [1, 2]       // && [1, 2) (PG range overlap (have points in common) operator)
[Op.adjacent]: [1, 2]      // -|- [1, 2) (PG range is adjacent to operator)
[Op.strictLeft]: [1, 2]    // << [1, 2) (PG range strictly left of operator)
[Op.strictRight]: [1, 2]   // >> [1, 2) (PG range strictly right of operator)
[Op.noExtendRight]: [1, 2] // &< [1, 2) (PG range does not extend to the right of operator)
[Op.noExtendLeft]: [1, 2]  // &> [1, 2) (PG range does not extend to the left of operator)
```

#### 组合
```js
{
  rank: {
    [Op.or]: {
      [Op.lt]: 1000,
      [Op.eq]: null
    }
  }
}
// rank < 1000 OR rank IS NULL

{
  createdAt: {
    [Op.lt]: new Date(),
    [Op.gt]: new Date(new Date() - 24 * 60 * 60 * 1000)
  }
}
// createdAt < [timestamp] AND createdAt > [timestamp]

{
  [Op.or]: [
    {
      title: {
        [Op.like]: 'Boat%'
      }
    },
    {
      description: {
        [Op.like]: '%boat%'
      }
    }
  ]
}
// title LIKE 'Boat%' OR description LIKE '%boat%'
```

#### 运算符别名

Sequelize 允许将特定字符串设置为操作符的别名 -

```js
const Op = Sequelize.Op;
const operatorsAliases = {
  $gt: Op.gt
}
const connection = new Sequelize(db, user, pass, { operatorsAliases })

[Op.gt]: 6 // > 6
$gt: 6 // 等同于使用 Op.gt (> 6)
```


#### 运算符安全性

使用没有任何别名的 Sequelize 可以提高安全性。

一些框架会自动将用户输入解析为js对象，如果您无法清理输入，则可能会将具有字符串运算符的对象注入到 Sequelize。

不带任何字符串别名将使运算符不太可能被注入，但您应该始终正确验证和清理用户输入。

由于向后兼容性原因Sequelize默认设置以下别名 -

$eq, $ne, $gte, $gt, $lte, $lt, $not, $in, $notIn, $is, $like, $notLike, $iLike, $notILike, $regexp, $notRegexp, $iRegexp, $notIRegexp, $between, $notBetween, $overlap, $contains, $contained, $adjacent, $strictLeft, $strictRight, $noExtendRight, $noExtendLeft, $and, $or, $any, $all, $values, $col

目前，以下遗留别名也被设置，但计划在不久的将来完全删除 -

ne, not, in, notIn, gte, gt, lte, lt, like, ilike, $ilike, nlike, $notlike, notilike, .., between, !.., notbetween, nbetween, overlap, &&, @>, <@

为了更好的安全性，建议使用 `Sequelize.Op`，而不是依赖任何字符串别名。 您可以通过设置`operatorsAliases`选项来限制应用程序需要的别名，请记住要清理用户输入，特别是当您直接将它们传递给 Sequelize 方法时。

```js
const Op = Sequelize.Op;

// 不用任何操作符别名使用 sequelize 
const connection = new Sequelize(db, user, pass, { operatorsAliases: false });

// 只用 $and => Op.and 操作符别名使用 sequelize 
const connection2 = new Sequelize(db, user, pass, { operatorsAliases: { $and: Op.and } });
```

如果你使用默认别名并且不限制它们，Sequelize会发出警告。如果您想继续使用所有默认别名（不包括旧版别名）而不发出警告，您可以传递以下运算符参数 -

```js
const Op = Sequelize.Op;
const operatorsAliases = {
  $eq: Op.eq,
  $ne: Op.ne,
  $gte: Op.gte,
  $gt: Op.gt,
  $lte: Op.lte,
  $lt: Op.lt,
  $not: Op.not,
  $in: Op.in,
  $notIn: Op.notIn,
  $is: Op.is,
  $like: Op.like,
  $notLike: Op.notLike,
  $iLike: Op.iLike,
  $notILike: Op.notILike,
  $regexp: Op.regexp,
  $notRegexp: Op.notRegexp,
  $iRegexp: Op.iRegexp,
  $notIRegexp: Op.notIRegexp,
  $between: Op.between,
  $notBetween: Op.notBetween,
  $overlap: Op.overlap,
  $contains: Op.contains,
  $contained: Op.contained,
  $adjacent: Op.adjacent,
  $strictLeft: Op.strictLeft,
  $strictRight: Op.strictRight,
  $noExtendRight: Op.noExtendRight,
  $noExtendLeft: Op.noExtendLeft,
  $and: Op.and,
  $or: Op.or,
  $any: Op.any,
  $all: Op.all,
  $values: Op.values,
  $col: Op.col
};

const connection = new Sequelize(db, user, pass, { operatorsAliases });
```

### JSON

JSON 数据类型仅由 PostgreSQL，SQLite 和 MySQL 语言支持。

#### PostgreSQL

PostgreSQL 中的 JSON 数据类型将值存储为纯文本，而不是二进制表示。 如果您只是想存储和检索 JSON 格式数据，那么使用 JSON 将占用更少的磁盘空间，并且从其输入数据中构建时间更少。 但是，如果您想对 JSON 值执行任何操作，则应该使用下面描述的 JSONB 数据类型。

#### MSSQL

MSSQL 没有 JSON 数据类型，但是它确实提供了对于自 SQL Server 2016 以来通过某些函数存储为字符串的 JSON 的支持。使用这些函数，您将能够查询存储在字符串中的 JSON，但是任何返回的值将需要分别进行解析。

```js
// ISJSON - 测试一个字符串是否包含有效的 JSON
User.findAll({
  where: sequelize.where(sequelize.fn('ISJSON', sequelize.col('userDetails')), 1)
})

// JSON_VALUE - 从 JSON 字符串提取标量值
User.findAll({
  attributes: [[ sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), 'address line 1']]
})

// JSON_VALUE - 从 JSON 字符串中查询标量值
User.findAll({
  where: sequelize.where(sequelize.fn('JSON_VALUE', sequelize.col('userDetails'), '$.address.Line1'), '14, Foo Street')
})

// JSON_QUERY - 提取一个对象或数组
User.findAll({
  attributes: [[ sequelize.fn('JSON_QUERY', sequelize.col('userDetails'), '$.address'), 'full address']]
})
```

### JSONB

JSONB 可以以三种不同的方式进行查询。

#### 嵌套对象
```js
{
  meta: {
    video: {
      url: {
        [Op.ne]: null
      }
    }
  }
}
```

#### 嵌套键
```js
{
  "meta.audio.length": {
    [Op.gt]: 20
  }
}
```

#### 外包裹
```js
{
  "meta": {
    [Op.contains]: {
      site: {
        url: 'http://google.com'
      }
    }
  }
}
```

### 关系 / 关联
```js
// 找到所有具有至少一个 task 的  project，其中 task.state === project.state
Project.findAll({
    include: [{
        model: Task,
        where: { state: Sequelize.col('project.state') }
    }]
})
```

## 分页 / 限制

```js
// 获取10个实例/行
Project.findAll({ limit: 10 })

// 跳过8个实例/行
Project.findAll({ offset: 8 })

// 跳过5个实例，然后取5个
Project.findAll({ offset: 5, limit: 5 })
```

## 排序

`order` 需要一个条目的数组来排序查询或者一个 sequelize 方法。一般来说，你将要使用任一属性的 tuple/array，并确定排序的正反方向。

```js
Subtask.findAll({
  order: [
    // 将转义用户名，并根据有效的方向参数列表验证DESC
    ['title', 'DESC'],

    // 将按最大值排序(age)
    sequelize.fn('max', sequelize.col('age')),

    // 将按最大顺序(age) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // 将按 otherfunction 排序(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // 将使用模型名称作为关联的名称排序关联模型的 created_at。
    [Task, 'createdAt', 'DESC'],

    // Will order through an associated model's created_at using the model names as the associations' names.
    [Task, Project, 'createdAt', 'DESC'],

    // 将使用关联的名称由关联模型的created_at排序。
    ['Task', 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using the names of the associations.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using an association object. (优选方法)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // Will order by a nested associated model's created_at using association objects. (优选方法)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // Will order by an associated model's created_at using a simple association object.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // 嵌套关联模型的 created_at 简单关联对象排序
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
  ]
  
  // 将按年龄最大值降序排列
  order: sequelize.literal('max(age) DESC')

  // 按最年龄大值升序排列，当省略排序条件时默认是升序排列
  order: sequelize.fn('max', sequelize.col('age'))

  // 按升序排列是省略排序条件的默认顺序
  order: sequelize.col('age')
  
  // 将根据方言随机排序 (而不是 fn('RAND') 或 fn('RANDOM'))
  order: sequelize.random()
})
```

## Table Hint

当使用 mssql 时，可以使用 `tableHint` 来选择传递一个表提示。 该提示必须是来自 `Sequelize.TableHints` 的值，只能在绝对必要时使用。 每个查询当前仅支持单个表提示。

表提示通过指定某些选项来覆盖 mssql 查询优化器的默认行为。 它们只影响该子句中引用的表或视图。

```js
const TableHints = Sequelize.TableHints;

Project.findAll({
  // 添加 table hint NOLOCK
  tableHint: TableHints.NOLOCK
  // 这将生成 SQL 'WITH (NOLOCK)'
})
```
