# Eager Loading - 预先加载

如[关联指南](../core-concepts/assocs.md)中简要提到的,预先加载是一次查询多个模型(一个"主"模型和一个或多个关联模型)的数据的行为. 在 SQL 级别上,这是具有一个或多个 [join](https://en.wikipedia.org/wiki/Join_(SQL)) 的查询.

完成此操作后,Sequelize 将在返回的对象中将适当关联的模型添加到适当命名的自动创建的字段中.

在 Sequelize 中,主要通过在模型查找器查询中使用 `include` 参数(例如,`findOne`, `findAll` 等)来完成预先加载.

## 基本示例

让我们假设以下设置：

```js
const User = sequelize.define('user', { name: DataTypes.STRING }, { timestamps: false });
const Task = sequelize.define('task', { name: DataTypes.STRING }, { timestamps: false });
const Tool = sequelize.define('tool', {
  name: DataTypes.STRING,
  size: DataTypes.STRING
}, { timestamps: false });
User.hasMany(Task);
Task.belongsTo(User);
User.hasMany(Tool, { as: 'Instruments' });
```

### 获取单个关联元素

首先,让我们用其关联的用户加载所有任务：

```js
const tasks = await Task.findAll({ include: User });
console.log(JSON.stringify(tasks, null, 2));
```

输出:

```json
[{
  "name": "A Task",
  "id": 1,
  "userId": 1,
  "user": {
    "name": "John Doe",
    "id": 1
  }
}]
```

这里,`tasks[0].user instanceof User` 是 `true`. 这表明,当 Sequelize 提取关联的模型时,它们将作为模型实例添加到输出对象.

上面,在获取的任务中,关联的模型被添加到名为 `user` 的新字段中. Sequelize 会根据关联模型的名称自动选择此字段的名称,在适用的情况下(即关联为 `hasMany` 或 `belongsToMany`)使用该字段的复数形式. 换句话说,由于`Task.belongsTo(User)`导致一项任务与一个用户相关联,因此逻辑选择是单数形式(Sequelize 自动遵循该形式).

### 获取所有关联的元素

现在,我们将执行相反的操作,而不是加载与给定任务关联的用户,我们将找到与给定用户关联的所有任务.

方法调用本质上是相同的. 唯一的区别是,现在在查询结果中创建的额外字段使用复数形式(在这种情况下为 `tasks`),其值是任务实例的数组(而不是上面的单个实例).

```js
const users = await User.findAll({ include: Task });
console.log(JSON.stringify(users, null, 2));
```

输出:

```json
[{
  "name": "John Doe",
  "id": 1,
  "tasks": [{
    "name": "A Task",
    "id": 1,
    "userId": 1
  }]
}]
```

注意,由于关联是一对多的,因此访问器(结果实例中的`tasks`属性)是复数的.

### 获取别名关联

如果关联是别名的(使用`as`参数),则在包含模型时必须指定此别名. 与其直接将模型传递给 `include` 参数,不如为对象提供两个选项：`model` 和 `as`.

注意上面的用户的 `Tool` 是如何被别名为 `Instruments` 的. 为了实现这一点,你必须指定要加载的模型以及别名：

```js
const users = await User.findAll({
  include: { model: Tool, as: 'Instruments' }
});
console.log(JSON.stringify(users, null, 2));
```

Output:

```json
[{
  "name": "John Doe",
  "id": 1,
  "Instruments": [{
    "name": "Scissor",
    "id": 1,
    "userId": 1
  }]
}]
```

你也可以包括指定的关联别名相匹配的字符串：

```js
User.findAll({ include: 'Instruments' }); // 也可以正常使用
User.findAll({ include: { association: 'Instruments' } }); // 也可以正常使用
```

### 需要预先加载

预先加载时,我们可以强制查询仅返回具有关联模型的记录,从而有效地将查询从默认的 `OUTER JOIN` 转为 `INNER JOIN`. 这是通过 `required: true` 参数完成的,如下所示：

```js
User.findAll({
  include: {
    model: Task,
    required: true
  }
});
```

此参数也适用于嵌套包含.

### 在模型级别的预先加载过滤

预先加载时,我们还可以使用 `where` 参数过滤关联的模型,如以下示例所示：

```js
User.findAll({
  include: {
    model: Tool,
    as: 'Instruments'
    where: {
      size: {
        [Op.ne]: 'small'
      }
    }
  }
});
```

生成 SQL:

```sql
SELECT
  `user`.`id`,
  `user`.`name`,
  `Instruments`.`id` AS `Instruments.id`,
  `Instruments`.`name` AS `Instruments.name`,
  `Instruments`.`size` AS `Instruments.size`,
  `Instruments`.`userId` AS `Instruments.userId`
FROM `users` AS `user`
INNER JOIN `tools` AS `Instruments` ON
  `user`.`id` = `Instruments`.`userId` AND
  `Instruments`.`size` != 'small';
```

请注意,上面生成的 SQL 查询将仅获取具有至少一个符合条件(在这种情况下为 `small`)的工具的用户. 出现这种情况是因为,当在 `include` 内使用 `where` 参数时,Sequelize 会自动将 `required` 参数设置为 `true`. 这意味着,将执行 `INNER JOIN` 而不是 `OUTER JOIN`,仅返回具有至少一个匹配子代的父代模型.

还要注意,使用的 `where` 参数已转换为 `INNER JOIN` 的 `ON` 子句的条件. 为了获得 *顶层* 的 `WHERE` 子句,而不是 `ON` 子句,必须做一些不同的事情.接下来将展示.

#### 参考其他列

如果你想在包含模型中应用 `WHERE` 子句来引用关联模型中的值,则可以简单地使用 `Sequelize.col` 函数,如以下示例所示：

```js
// 查找所有具有至少一项任务的项目,其中 task.state === project.state
Project.findAll({
  include: {
    model: Task,
    where: {
      state: Sequelize.col('project.state')
    }
  }
})
```

### 顶层的复杂 where 子句

为了获得涉及嵌套列的顶级 `WHERE` 子句,Sequelize 提供了一种引用嵌套列的方法：`'$nested.column$'` 语法.

例如,它可以用于将 `where` 条件从包含的模型从 `ON` 条件移动到顶层的 `WHERE` 子句.

```js
User.findAll({
  where: {
    '$Instruments.size$': { [Op.ne]: 'small' }
  },
  include: [{
    model: Tool,
    as: 'Instruments'
  }]
});
```

生成 SQL:

```sql
SELECT
  `user`.`id`,
  `user`.`name`,
  `Instruments`.`id` AS `Instruments.id`,
  `Instruments`.`name` AS `Instruments.name`,
  `Instruments`.`size` AS `Instruments.size`,
  `Instruments`.`userId` AS `Instruments.userId`
FROM `users` AS `user`
LEFT OUTER JOIN `tools` AS `Instruments` ON
  `user`.`id` = `Instruments`.`userId`
WHERE `Instruments`.`size` != 'small';
```

`$nested.column$` 语法也适用于嵌套了多个级别的列,例如 `$some.super.deeply.nested.column$`. 因此,你可以使用它对深层嵌套的列进行复杂的过滤.

为了更好地理解内部的 `where` 参数(在 `include` 内部使用)和使用与不使用 `required` 参数与使用 `$nested.column$`  语法的顶级 `where` 之间的所有区别. ,下面我们为你提供四个示例：

```js
// Inner where, 默认使用 `required: true`
await User.findAll({
  include: {
    model: Tool,
    as: 'Instruments',
    where: {
      size: { [Op.ne]: 'small' }
    }
  }
});

// Inner where, `required: false`
await User.findAll({
  include: {
    model: Tool,
    as: 'Instruments',
    where: {
      size: { [Op.ne]: 'small' }
    },
    required: false
  }
});

// 顶级 where, 默认使用 `required: false`
await User.findAll({
  where: {
    '$Instruments.size$': { [Op.ne]: 'small' }
  },
  include: {
    model: Tool,
    as: 'Instruments'
  }
});

// 顶级 where, `required: true`
await User.findAll({
  where: {
    '$Instruments.size$': { [Op.ne]: 'small' }
  },
  include: {
    model: Tool,
    as: 'Instruments',
    required: true
  }
});
```

生成 SQL:

```sql
-- Inner where, 默认使用 `required: true`
SELECT [...] FROM `users` AS `user`
INNER JOIN `tools` AS `Instruments` ON
  `user`.`id` = `Instruments`.`userId`
  AND `Instruments`.`size` != 'small';

-- Inner where, `required: false`
SELECT [...] FROM `users` AS `user`
LEFT OUTER JOIN `tools` AS `Instruments` ON
  `user`.`id` = `Instruments`.`userId`
  AND `Instruments`.`size` != 'small';

-- 顶级 where, 默认使用 `required: false`
SELECT [...] FROM `users` AS `user`
LEFT OUTER JOIN `tools` AS `Instruments` ON
  `user`.`id` = `Instruments`.`userId`
WHERE `Instruments`.`size` != 'small';

-- 顶级 where, `required: true`
SELECT [...] FROM `users` AS `user`
INNER JOIN `tools` AS `Instruments` ON
  `user`.`id` = `Instruments`.`userId`
WHERE `Instruments`.`size` != 'small';
```

### 使用 `RIGHT OUTER JOIN` 获取 (仅限 MySQL, MariaDB, PostgreSQL 和 MSSQL)

默认情况下,关联是使用 `LEFT OUTER JOIN` 加载的 - 也就是说,它仅包含来自父表的记录. 如果你使用的方言支持,你可以通过传递 `right` 选项来将此行为更改为 `RIGHT OUTER JOIN`.

当前, SQLite 不支持 [right joins](https://www.sqlite.org/omitted.html).

*注意:* 仅当 `required` 为 false 时才遵循 `right`.

```js
User.findAll({
  include: [{
    model: Task // 将创建一个 left join
  }]
});
User.findAll({
  include: [{
    model: Task,
    right: true // 将创建一个 right join
  }]
});
User.findAll({
  include: [{
    model: Task,
    required: true,
    right: true // 没有效果, 将创建一个 inner join
  }]
});
User.findAll({
  include: [{
    model: Task,
    where: { name: { [Op.ne]: 'empty trash' } },
    right: true // 没有效果, 将创建一个 inner join
  }]
});
User.findAll({
  include: [{
    model: Tool,
    where: { name: { [Op.ne]: 'empty trash' } },
    required: false // 将创建一个 left join
  }]
});
User.findAll({
  include: [{
    model: Tool,
    where: { name: { [Op.ne]: 'empty trash' } },
    required: false
    right: true // 将创建一个 right join
  }]
});
```

## 多次预先加载

`include` 参数可以接收一个数组,以便一次获取多个关联的模型：

```js
Foo.findAll({
  include: [
    {
      model: Bar,
      required: true
    },
    {
      model: Baz,
      where: /* ... */
    },
    Qux // { model: Qux } 的简写语法在这里也适用
  ]
})
```

## 多对多关系的预先加载

当你对具有 "多对多" 关系的模型执行预先加载时,默认情况下,Sequelize 也将获取联结表数据. 例如：

```js
const Foo = sequelize.define('Foo', { name: DataTypes.TEXT });
const Bar = sequelize.define('Bar', { name: DataTypes.TEXT });
Foo.belongsToMany(Bar, { through: 'Foo_Bar' });
Bar.belongsToMany(Foo, { through: 'Foo_Bar' });

await sequelize.sync();
const foo = await Foo.create({ name: 'foo' });
const bar = await Bar.create({ name: 'bar' });
await foo.addBar(bar);
const fetchedFoo = Foo.findOne({ include: Bar });
console.log(JSON.stringify(fetchedFoo, null, 2));
```

输出:

```json
{
  "id": 1,
  "name": "foo",
  "Bars": [
    {
      "id": 1,
      "name": "bar",
      "Foo_Bar": {
        "FooId": 1,
        "BarId": 1
      }
    }
  ]
}
```

请注意,每个预先加载到 `Bars` 属性中的 bar 实例都有一个名为 `Foo_Bar` 的额外属性,它是联结模型的相关 Sequelize 实例. 默认情况下,Sequelize 从联结表中获取所有属性,以构建此额外属性.

然而,你可以指定要获取的属性. 这是通过在包含的 `through` 参数中应用 `attributes` 参数来完成的. 例如：

```js
Foo.findAll({
  include: [{
    model: Bar,
    through: {
      attributes: [/* 在此处列出所需的属性 */]
    }
  }]
});
```

如果你不需要联结表中的任何内容,则可以显式地为 `attributes` 参数提供一个空数组,在这种情况下,将不会获取任何内容,甚至不会创建额外的属性：

```js
Foo.findOne({
  include: {
    model: Bar,
    attributes: []
  }
});
```

输出:

```json
{
  "id": 1,
  "name": "foo",
  "Bars": [
    {
      "id": 1,
      "name": "bar"
    }
  ]
}
```

每当包含 "多对多" 关系中的模型时,也可以在联结表上应用过滤器. 这是通过在 `include` 的 `through` 参数中应用 `where` 参数来完成的. 例如：

```js
User.findAll({
  include: [{
    model: Project,
    through: {
      where: {
        // 这里,`completed` 是联结表上的一列
        completed: true
      }
    }
  }]
});
```

生成 SQL (使用 SQLite):

```sql
SELECT
  `User`.`id`,
  `User`.`name`,
  `Projects`.`id` AS `Projects.id`,
  `Projects`.`name` AS `Projects.name`,
  `Projects->User_Project`.`completed` AS `Projects.User_Project.completed`,
  `Projects->User_Project`.`UserId` AS `Projects.User_Project.UserId`,
  `Projects->User_Project`.`ProjectId` AS `Projects.User_Project.ProjectId`
FROM `Users` AS `User`
LEFT OUTER JOIN `User_Projects` AS `Projects->User_Project` ON
  `User`.`id` = `Projects->User_Project`.`UserId`
LEFT OUTER JOIN `Projects` AS `Projects` ON
  `Projects`.`id` = `Projects->User_Project`.`ProjectId` AND
  `Projects->User_Project`.`completed` = 1;
```

## 包括一切

要包括所有关联的模型,可以使用 `all` 和 `nested` 参数：

```js
// 提取与用户关联的所有模型
User.findAll({ include: { all: true }});

// 递归获取与用户及其嵌套关联关联的所有模型
User.findAll({ include: { all: true, nested: true }});
```

## 包括软删除的记录

如果你想加载软删除的记录,可以通过将 `include.paranoid` 设置为 `false` 来实现：

```js
User.findAll({
  include: [{
    model: Tool,
    as: 'Instruments',
    where: { size: { [Op.ne]: 'small' } },
    paranoid: false
  }]
});
```

## 排序预先加载的关联

当你想将 `ORDER` 子句应用于预先加载的模型时,必须对扩展数组使用顶层 `order` 参数,从要排序的嵌套模型开始.

通过示例可以更好地理解这一点.

```js
Company.findAll({
  include: Division,
  order: [
    // 我们从要排序的模型开始排序数组
    [Division, 'name', 'ASC']
  ]
});
Company.findAll({
  include: Division,
  order: [
    [Division, 'name', 'DESC']
  ]
});
Company.findAll({
  // 如果包含使用别名...
  include: { model: Division, as: 'Div' },
  order: [
    // ...我们在排序数组的开头使用来自 `include` 的相同语法
    [{ model: Division, as: 'Div' }, 'name', 'DESC']
  ]
});

Company.findAll({
  // 如果我们包含嵌套在多个级别中...
  include: {
    model: Division,
    include: Department
  },
  order: [
    // ... 我们在排序数组的开头复制需要的 include 链
    [Division, Department, 'name', 'DESC']
  ]
});
```

对于多对多关系,你还可以按联结表中的属性进行排序.例如,假设我们在 `Division` 和 `Department` 之间存在多对多关系,联结模型为 `DepartmentDivision`, 你可以这样:

```js
Company.findAll({
  include: {
    model: Division,
    include: Department
  },
  order: [
    [Division, DepartmentDivision, 'name', 'ASC']
  ]
});
```

在以上所有示例中,你已经注意到在顶层使用了 `order` 参数. 但是在`separate: true` 时,`order` 也可以在 `include` 参数中使用. 在这种情况下,用法如下：

```js
// 这仅能用于 `separate: true` (反过来仅适用于 HasMany 关系).
User.findAll({
  include: {
    model: Post,
    separate: true,
    order: [
      ['createdAt', 'DESC']
    ]
  }
});
```

### 涉及子查询的复杂排序

查看[子查询指南](../other-topics/sub-queries.md)上的示例,了解如何使用子查询来协助更复杂的排序.

## 嵌套的预先加载

你可以使用嵌套的预先加载来加载相关模型的所有相关模型：

```js
const users = await User.findAll({
  include: {
    model: Tool,
    as: 'Instruments',
    include: {
      model: Teacher,
      include: [ /* ... */ ]
    }
  }
});
console.log(JSON.stringify(users, null, 2));
```

输出:

```json
[{
  "name": "John Doe",
  "id": 1,
  "Instruments": [{ // 1:M 和 N:M 关联
    "name": "Scissor",
    "id": 1,
    "userId": 1,
    "Teacher": { // 1:1 关联
      "name": "Jimi Hendrix"
    }
  }]
}]
```

这将产生一个外部连接. 但是,相关模型上的 `where` 子句将创建内部联接,并且仅返回具有匹配子模型的实例. 要返回所有父实例,你应该添加 `required: false`.

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
});
```

上面的查询将返回所有用户及其所有乐器,但仅返回与 `Woodstock Music School` 相关的那些老师.

## 使用带有 include 的 `findAndCountAll`

`findAndCountAll` 实用功能支持 include. 仅将标记为 `required` 的 include 项视为 `count`. 例如,如果你要查找并统计所有拥有个人资料的用户：

```js
User.findAndCountAll({
  include: [
    { model: Profile, required: true }
  ],
  limit: 3
});
```

因为 `Profile` 的 include 已设置为 `required`,它将导致内部联接,并且仅统计具有个人资料的用户. 如果我们从包含中删除 `required`,则包含和不包含配置文件的用户都将被计数. 在 include 中添加一个 `where` 子句会自动使它成为 required：

```js
User.findAndCountAll({
  include: [
    { model: Profile, where: { active: true } }
  ],
  limit: 3
});
```

上面的查询仅会统计拥有有效个人资料的用户,因为当你在 `include` 中添加 `where` 子句时,`required` 会隐式设置为 `true`.