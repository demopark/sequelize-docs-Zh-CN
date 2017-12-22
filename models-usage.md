# Model usage - 模型使用

## 数据检索/查找器

Finder 方法旨在从数据库查询数据。 他们 **不** 返回简单的对象，而是返回模型实例。 因为 finder 方法返回模型实例，您可以按照  [*实例*](/manual/tutorial/instances.html) 的文档中所述，为结果调用任何模型实例成员。
 
在本文中，我们将探讨 finder 方法可以做什么:

### `find` - 搜索数据库中的一个特定元素
```js
// 搜索已知的ids
Project.findById(123).then(project => {
  // project 将是 Project的一个实例，并具有在表中存为 id 123 条目的内容。
  // 如果没有定义这样的条目，你将获得null
})

// 搜索属性
Project.findOne({ where: {title: 'aProject'} }).then(project => {
  // project 将是 Projects 表中 title 为 'aProject'  的第一个条目 || null
})


Project.findOne({
  where: {title: 'aProject'},
  attributes: ['id', ['name', 'title']]
}).then(project => {
  // project 将是 Projects 表中 title 为 'aProject'  的第一个条目 || null
  // project.title 将包含 project 的 name
})
```

### `findOrCreate` - 搜索特定元素或创建它（如果不可用）

方法 `findOrCreate` 可用于检查数据库中是否已存在某个元素。 如果是这种情况，则该方法将生成相应的实例。 如果元素不存在，将会被创建。

如果是这种情况，则该方法将导致相应的实例。 如果元素不存在，将会被创建。

假设我们有一个空的数据库，一个 `User` 模型有一个 `username` 和 `job`。

```js
User
  .findOrCreate({where: {username: 'sdepold'}, defaults: {job: 'Technical Lead JavaScript'}})
  .spread((user, created) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
    findOrCreate 返回一个包含已找到或创建的对象的数组，找到或创建的对象和一个布尔值，如果创建一个新对象将为true，否则为false，像这样:

    [ {
        username: 'sdepold',
        job: 'Technical Lead JavaScript',
        id: 1,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      true ]

在上面的例子中，".spread" 将数组分成2部分，并将它们作为参数传递给回调函数，在这种情况下将它们视为 "user" 和 "created" 。（所以“user”将是返回数组的索引0的对象，并且 "created" 将等于 "true"。）
    */
  })
```

代码创建了一个新的实例。 所以当我们已经有一个实例了 ...

```js
User.create({ username: 'fnord', job: 'omnomnom' })
  .then(() => User.findOrCreate({where: {username: 'fnord'}, defaults: {job: 'something else'}}))
  .spread((user, created) => {
    console.log(user.get({
      plain: true
    }))
    console.log(created)

    /*
    在这个例子中，findOrCreate 返回一个如下的数组：
    [ {
        username: 'fnord',
        job: 'omnomnom',
        id: 2,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      false
    ]
    由findOrCreate返回的数组通过 ".spread" 扩展为两部分，并且这些部分将作为2个参数传递给回调函数，在这种情况下将其视为 "user" 和 "created" 。（所以“user”将是返回数组的索引0的对象，并且 "created" 将等于 "false"。）
    */
  })
```

...现有条目将不会更改。 看到第二个用户的 "job"，并且实际上创建操作是假的。

### `findAndCountAll` - 在数据库中搜索多个元素，返回数据和总计数

这是一个方便的方法，它结合了 `findAll` 和 `count`（见下文），当处理与分页相关的查询时，这是有用的，你想用 `limit` 和 `offset` 检索数据，但也需要知道总数与查询匹配的记录数：

处理程序成功将始终接收具有两个属性的对象：

* `count` - 一个整数，总数记录匹配where语句和关联的其它过滤器
* `rows` - 一个数组对象，记录在limit和offset范围内匹配where语句和关联的其它过滤器，

```js
Project
  .findAndCountAll({
     where: {
        title: {
          [Op.like]: 'foo%'
        }
     },
     offset: 10,
     limit: 2
  })
  .then(result => {
    console.log(result.count);
    console.log(result.rows);
  });
```

它支持 include。 只有标记为 `required` 的 include 将被添加到计数部分：

假设您想查找附有个人资料的所有用户：

```js
User.findAndCountAll({
  include: [
     { model: Profile, required: true}
  ],
  limit: 3
});
```

因为 `Profile` 的 include 有 `required` 设置，这将导致内部连接，并且只有具有 profile 的用户将被计数。 如果我们从 include 中删除`required`，那么有和没有 profile 的用户都将被计数。 在include中添加一个 `where` 语句会自动使它成为 required：

```js
User.findAndCountAll({
  include: [
     { model: Profile, where: { active: true }}
  ],
  limit: 3
});
```

上面的查询只会对具有 active profile 的用户进行计数，因为在将 where 语句添加到 include 时，`required` 被隐式设置为 true。

传递给 `findAndCountAll` 的 options 对象与 `findAll` 相同（如下所述）。

### `findAll` - 搜索数据库中的多个元素
```js
// 找到多个条目
Project.findAll().then(projects => {
  // projects 将是所有 Project 实例的数组
})

// 也可以：
Project.all().then(projects => {
  // projects 将是所有 Project 实例的数组
})

// 搜索特定属性 - 使用哈希
Project.findAll({ where: { name: 'A Project' } }).then(projects => {
  // projects将是一个具有指定 name 的 Project 实例数组
})

// 在特定范围内进行搜索
Project.findAll({ where: { id: [1,2,3] } }).then(projects => {
  // projects将是一系列具有 id 1,2 或 3 的项目
  // 这实际上是在做一个 IN 查询
})

Project.findAll({
  where: {
    id: {
      [Op.and]: {a: 5},           // 且 (a = 5)
      [Op.or]: [{a: 5}, {a: 6}],  // (a = 5 或 a = 6)
      [Op.gt]: 6,                // id > 6
      [Op.gte]: 6,               // id >= 6
      [Op.lt]: 10,               // id < 10
      [Op.lte]: 10,              // id <= 10
      [Op.ne]: 20,               // id != 20
      [Op.between]: [6, 10],     // 在 6 和 10 之间
      [Op.notBetween]: [11, 15], // 不在 11 和 15 之间
      [Op.in]: [1, 2],           // 在 [1, 2] 之中
      [Op.notIn]: [1, 2],        // 不在 [1, 2] 之中
      [Op.like]: '%hat',         // 包含 '%hat'
      [Op.notLike]: '%hat',       // 不包含 '%hat'
      [Op.iLike]: '%hat',         // 包含 '%hat' (不区分大小写)  (仅限 PG)
      [Op.notILike]: '%hat',      // 不包含 '%hat'  (仅限 PG)
      [Op.overlap]: [1, 2],       // && [1, 2] (PG数组重叠运算符)
      [Op.contains]: [1, 2],      // @> [1, 2] (PG数组包含运算符)
      [Op.contained]: [1, 2],     // <@ [1, 2] (PG数组包含于运算符)
      [Op.any]: [2,3],            // 任何数组[2, 3]::INTEGER (仅限 PG)
    },
    status: {
      [Op.not]: false,           // status 不为 FALSE
    }
  }
})
```

### 复合过滤 / OR / NOT 查询

你可以使用多层嵌套的 AND，OR 和 NOT 条件进行一个复合的 where 查询。 为了做到这一点，你可以使用  `or` ， `and` 或 `not` `运算符`:


```js
Project.findOne({
  where: {
    name: 'a project',
    [Op.or]: [
      { id: [1,2,3] },
      { id: { [Op.gt]: 10 } }
    ]
  }
})

Project.findOne({
  where: {
    name: 'a project',
    id: {
      [Op.or]: [
        [1,2,3],
        { [Op.gt]: 10 }
      ]
    }
  }
})
```

这两段代码将生成以下内容：

```sql
SELECT *
FROM `Projects`
WHERE (
  `Projects`.`name` = 'a project'
   AND (`Projects`.`id` IN (1,2,3) OR `Projects`.`id` > 10)
)
LIMIT 1;
```

`not` 示例:

```js
Project.findOne({
  where: {
    name: 'a project',
    [Op.not]: [
      { id: [1,2,3] },
      { array: { [Op.contains]: [3,4,5] } }
    ]
  }
});
```

将生成:

```sql
SELECT *
FROM `Projects`
WHERE (
  `Projects`.`name` = 'a project'
   AND NOT (`Projects`.`id` IN (1,2,3) OR `Projects`.`array` @> ARRAY[3,4,5]::INTEGER[])
)
LIMIT 1;
```

### 用限制，偏移，顺序和分组操作数据集

要获取更多相关数据，可以使用限制，偏移，顺序和分组：

```js
// 限制查询的结果
Project.findAll({ limit: 10 })

// 跳过前10个元素
Project.findAll({ offset: 10 })

// 跳过前10个元素，并获取2个
Project.findAll({ offset: 10, limit: 2 })
```

分组和排序的语法是相同的，所以下面只用一个单独的例子来解释分组，而其余的则是排序。 您下面看到的所有内容也可以对分组进行

```js
Project.findAll({order: 'title DESC'})
// 生成 ORDER BY title DESC

Project.findAll({group: 'name'})
// 生成 GROUP BY name
```

请注意，在上述两个示例中，提供的字符串逐字插入到查询中，所以不会转义列名称。 当你向 order / group 提供字符串时，将始终如此。 如果要转义列名，您应该提供一个参数数组，即使您只想通过单个列进行 order / group

```js
something.findOne({
  order: [
    // 将返回 `name`
    ['name'],
    // 将返回 `username` DESC
    ['username', 'DESC'],
    // 将返回 max(`age`)
    sequelize.fn('max', sequelize.col('age')),
    // 将返回 max(`age`) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],
    // 将返回 otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],
    // 将返回 otherfunction(awesomefunction(`col`)) DESC，这个嵌套是可以无限的！
    [sequelize.fn('otherfunction', sequelize.fn('awesomefunction', sequelize.col('col'))), 'DESC']
  ]
})
```

回顾一下，order / group数组的元素可以是以下内容：

* String - 将被引用
* Array - 第一个元素将被引用，第二个将被逐字地追加
* Object -
  * raw 将被添加逐字引用
  * 如果未设置 raw，一切都被忽略，查询将失败
* Sequelize.fn 和 Sequelize.col 返回函数和引用的列名

### 原始查询

有时候，你可能会期待一个你想要显示的大量数据集，而无需操作。 对于你选择的每一行，Sequelize 创建一个具有更新，删除和获取关联等功能的实例。如果您有数千行，则可能需要一些时间。 如果您只需要原始数据，并且不想更新任何内容，您可以这样做来获取原始数据。

```js
// 你期望从数据库的一个巨大的数据集,
// 并且不想花时间为每个条目构建DAO？
// 您可以传递一个额外的查询参数来取代原始数据：
Project.findAll({ where: { ... }, raw: true })
```

### `count` - 计算数据库中元素的出现次数

还有一种数据库对象计数的方法：

```js
Project.count().then(c => {
  console.log("There are " + c + " projects!")
})

Project.count({ where: {'id': {[Op.gt]: 25}} }).then(c => {
  console.log("There are " + c + " projects with an id greater than 25.")
})
```

### `max` - 获取特定表中特定属性的最大值

这里是获取属性的最大值的方法：

```js
/*
   我们假设3个具有属性年龄的对象。
   第一个是10岁，
   第二个是5岁，
   第三个是40岁。
*/
Project.max('age').then(max => {
  // 将返回 40
})

Project.max('age', { where: { age: { [Op.lt]: 20 } } }).then(max => {
  // 将会是 10
})
```

### `min` - 获取特定表中特定属性的最小值

这里是获取属性的最小值的方法：

```js
/*
   我们假设3个具有属性年龄的对象。
   第一个是10岁，
   第二个是5岁，
   第三个是40岁。
*/
Project.min('age').then(min => {
  // 将返回 5
})

Project.min('age', { where: { age: { [Op.gt]: 5 } } }).then(min => {
  // 将会是 10
})
```

### `sum` - 特定属性的值求和

为了计算表的特定列的总和，可以使用“sum”方法。

```js
/*
   我们假设3个具有属性年龄的对象。
   第一个是10岁，
   第二个是5岁，
   第三个是40岁。
*/
Project.sum('age').then(sum => {
  // 将返回 55
})

Project.sum('age', { where: { age: { [Op.gt]: 5 } } }).then(sum => {
  // 将会是 50
})
```

## 预加载

当你从数据库检索数据时，也想同时获得与之相关联的查询，这被称为预加载。这个基本思路就是当你调用 `find` 或 `findAll` 时使用  `include` 属性。让我们假设以下设置：

```js
const User = sequelize.define('user', { name: Sequelize.STRING })
const Task = sequelize.define('task', { name: Sequelize.STRING })
const Tool = sequelize.define('tool', { name: Sequelize.STRING })

Task.belongsTo(User)
User.hasMany(Task)
User.hasMany(Tool, { as: 'Instruments' })

sequelize.sync().then(() => {
  // 这是我们继续的地方 ...
})
```

首先，让我们用它们的关联 user 加载所有的 task。

```js
Task.findAll({ include: [ User ] }).then(tasks => {
  console.log(JSON.stringify(tasks))

  /*
    [{
      "name": "A Task",
      "id": 1,
      "createdAt": "2013-03-20T20:31:40.000Z",
      "updatedAt": "2013-03-20T20:31:40.000Z",
      "userId": 1,
      "user": {
        "name": "John Doe",
        "id": 1,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z"
      }
    }]
  */
})
```

请注意，访问者（结果实例中的 `User` 属性）是单数形式，因为关联是一对一的。

接下来的事情：用多对一的关联加载数据！

```js
User.findAll({ include: [ Task ] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "tasks": [{
        "name": "A Task",
        "id": 1,
        "createdAt": "2013-03-20T20:31:40.000Z",
        "updatedAt": "2013-03-20T20:31:40.000Z",
        "userId": 1
      }]
    }]
  */
})
```

请注意，访问者（结果实例中的  `Tasks`  属性）是复数形式，因为关联是多对一的。

如果关联是别名的（使用 `as` 参数），则在包含模型时必须指定此别名。 注意用户的  `Tool` 如何被别名为 `Instruments`。 为了获得正确的权限，您必须指定要加载的模型以及别名：

```js
User.findAll({ include: [{ model: Tool, as: 'Instruments' }] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1
      }]
    }]
  */
})
```

您还可以通过指定与关联别名匹配的字符串来包含别名：

```js
User.findAll({ include: ['Instruments'] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1
      }]
    }]
  */
})

User.findAll({ include: [{ association: 'Instruments' }] }).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1
      }]
    }]
  */
})
```

当预加载时，我们也可以使用 `where` 过滤关联的模型。 这将返回 `Tool` 模型中所有与 `where` 语句匹配的行的`User`。

```js
User.findAll({
    include: [{
        model: Tool,
        as: 'Instruments',
        where: { name: { [Op.like]: '%ooth%' } }
    }]
}).then(users => {
    console.log(JSON.stringify(users))

    /*
      [{
        "name": "John Doe",
        "id": 1,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],

      [{
        "name": "John Smith",
        "id": 2,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],
    */
  })
```

当使用 `include.where` 过滤一个预加载的模型时，`include.required` 被隐式设置为 `true`。 这意味着内部联接完成返回具有任何匹配子项的父模型。

### 使用预加载模型的顶层 WHERE

将模型的 `WHERE` 条件从 `ON` 条件的 include 模式移动到顶层，你可以使用 `'$nested.column$'` 语法：

```js
User.findAll({
    where: {
        '$Instruments.name$': { [Op.iLike]: '%ooth%' }
    },
    include: [{
        model: Tool,
        as: 'Instruments'
    }]
}).then(users => {
    console.log(JSON.stringify(users));

    /*
      [{
        "name": "John Doe",
        "id": 1,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],

      [{
        "name": "John Smith",
        "id": 2,
        "createdAt": "2013-03-20T20:31:45.000Z",
        "updatedAt": "2013-03-20T20:31:45.000Z",
        "Instruments": [{
          "name": "Toothpick",
          "id": 1,
          "createdAt": null,
          "updatedAt": null,
          "userId": 1
        }]
      }],
    */
```

### 包括所有

要包含所有属性，您可以使用 `all：true` 传递单个对象：

```js
User.findAll({ include: [{ all: true }]});
```

### 包括软删除的记录

如果想要加载软删除的记录，可以通过将 `include.paranoid` 设置为 `false` 来实现

```js
User.findAll({
    include: [{
        model: Tool,
        where: { name: { [Op.like]: '%ooth%' } },
        paranoid: false // query and loads the soft deleted records
    }]
});
```

### 排序预加载关联

在一对多关系的情况下。

```js
Company.findAll({ include: [ Division ], order: [ [ Division, 'name' ] ] });
Company.findAll({ include: [ Division ], order: [ [ Division, 'name', 'DESC' ] ] });
Company.findAll({
  include: [ { model: Division, as: 'Div' } ],
  order: [ [ { model: Division, as: 'Div' }, 'name' ] ]
});
Company.findAll({
  include: [ { model: Division, as: 'Div' } ],
  order: [ [ { model: Division, as: 'Div' }, 'name', 'DESC' ] ]
});
Company.findAll({
  include: [ { model: Division, include: [ Department ] } ],
  order: [ [ Division, Department, 'name' ] ]
});
```

在多对多关系的情况下，您还可以通过表中的属性进行排序。

```js
Company.findAll({
  include: [ { model: Division, include: [ Department ] } ],
  order: [ [ Division, DepartmentDivision, 'name' ] ]
});
```

### 嵌套预加载

您可以使用嵌套的预加载来加载相关模型的所有相关模型：

```js
User.findAll({
  include: [
    {model: Tool, as: 'Instruments', include: [
      {model: Teacher, include: [ /* etc */]}
    ]}
  ]
}).then(users => {
  console.log(JSON.stringify(users))

  /*
    [{
      "name": "John Doe",
      "id": 1,
      "createdAt": "2013-03-20T20:31:45.000Z",
      "updatedAt": "2013-03-20T20:31:45.000Z",
      "Instruments": [{ // 1:M and N:M association
        "name": "Toothpick",
        "id": 1,
        "createdAt": null,
        "updatedAt": null,
        "userId": 1,
        "Teacher": { // 1:1 association
          "name": "Jimi Hendrix"
        }
      }]
    }]
  */
})
```

这将产生一个外连接。 但是，相关模型上的 `where` 语句将创建一个内部连接，并仅返回具有匹配子模型的实例。 要返回所有父实例，您应该添加 `required: false`。

```js
User.findAll({
  include: [{
    model: Tool,
    as: 'Instruments',
    include: [{
      model: Teacher,
      where: {
        school: "Woodstock Music School"
      },
      required: false
    }]
  }]
}).then(users => {
  /* ... */
})
```

以上查询将返回所有用户及其所有乐器，但只会返回与 `Woodstock Music School` 相关的老师。

包括所有也支持嵌套加载：

```js
User.findAll({ include: [{ all: true, nested: true }]});
```
