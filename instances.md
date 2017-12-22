# Instances - 实例

## 构建非持久性实例

为了创建定义类的实例，请执行以下操作。 如果你以前编写过 Ruby，你可能认识该语法。 使用 `build` - 该方法将返回一个未保存的对象，你要明确地保存它。

```js
const project = Project.build({
  title: 'my awesome project',
  description: 'woot woot. this will make me a rich man'
})
 
const task = Task.build({
  title: 'specify the project idea',
  description: 'bla',
  deadline: new Date()
})
```

内置实例在定义时会自动获取默认值：

```js
// 首先定义模型
const Task = sequelize.define('task', {
  title: Sequelize.STRING,
  rating: { type: Sequelize.STRING, defaultValue: 3 }
})
 
// 现在实例化一个对象
const task = Task.build({title: 'very important task'})
 
task.title  // ==> 'very important task'
task.rating // ==> 3
```

要将其存储在数据库中，请使用 `save` 方法并捕获事件(如果需要):

```js
project.save().then(() => {
  // 回调
})
 
task.save().catch(error => {
  // 呃
})
 
// 还可以使用链式构建来保存和访问对象：
Task
  .build({ title: 'foo', description: 'bar', deadline: new Date() })
  .save()
  .then(anotherTask => {
    // 您现在可以使用变量 anotherTask 访问当前保存的任务
  })
  .catch(error => {
    // Ooops，做一些错误处理
  })
```

## 创建持久性实例

除了构建对象之外，还需要一个明确的保存调用来存储在数据库中，可以通过一个命令执行所有这些步骤。 它被称为 `create`。

```js
Task.create({ title: 'foo', description: 'bar', deadline: new Date() }).then(task => {
  // 你现在可以通过变量 task 来访问新创建的 task
})
```

也可以通过 `create` 方法定义哪些属性可以设置。 如果你创建基于可由用户填写的表单的数据库条目，这将非常方便。 例如，使用这种方式，你可以限制 `User` 模型，仅设置 username 和 address，而不是 admin 标志：

```js
User.create({ username: 'barfooz', isAdmin: true }, { fields: [ 'username' ] }).then(user => {
  // 我们假设 isAdmin 的默认值为 false：
  console.log(user.get({
    plain: true
  })) // => { username: 'barfooz', isAdmin: false }
})
```

## 更新 / 保存 / 持久化一个实例

现在可以更改一些值并将更改保存到数据库...有两种方法可以实现：

```js
// 方法 1
task.title = 'a very different title now'
task.save().then(() => {})
 
// 方法 2
task.update({
  title: 'a very different title now'
}).then(() => {})
```

通过传递列名数组，调用 `save` 时也可以定义哪些属性应该被保存。 当您基于先前定义的对象设置属性时，这是有用的。 例如。 如果您通过Web应用程序的形式获取对象的值。 此外，这在 `update` 内部使用。 它就像这样：

```js
task.title = 'foooo'
task.description = 'baaaaaar'
task.save({fields: ['title']}).then(() => {
 // title 现在将是 “foooo”，而 description 与以前一样
})
 
// 使用等效的 update 调用如下所示:
task.update({ title: 'foooo', description: 'baaaaaar'}, {fields: ['title']}).then(() => {
 //  title 现在将是 “foooo”，而 description 与以前一样
})
```

当你调用 `save `而不改变任何属性的时候，这个方法什么都不执行。 

## 销毁 / 删除持久性实例

创建对象并获得对象的引用后，可以从数据库中删除它。 相关的方法是 `destroy`：

```js
Task.create({ title: 'a task' }).then(task => {
  // 获取到 task 对象...
  return task.destroy();
}).then(() => {
 // task 对象已被销毁
})
```

如果 `paranoid` 选项为 true，则不会删除该对象，而将 `deletedAt` 列设置为当前时间戳。 要强制删除，可以将 `force: true` 传递给 destroy 调用：

```js
task.destroy({ force: true })
```

## 批量操作（一次创建，更新和销毁多行）

除了更新单个实例之外，你还可以一次创建，更新和删除多个实例。 调用你需要的方法

* `Model.bulkCreate`
* `Model.update`
* `Model.destroy`

由于你使用多个模型，回调将不会返回DAO实例。 BulkCreate将返回一个模型实例/DAO的数组，但是它们不同于`create`，没有 autoIncrement 属性的结果值. `update` 和 `destroy` 将返回受影响的行数。

首先看下 bulkCreate

```js
User.bulkCreate([
  { username: 'barfooz', isAdmin: true },
  { username: 'foo', isAdmin: true },
  { username: 'bar', isAdmin: false }
]).then(() => { // 注意: 这里没有凭据, 然而现在你需要...
  return User.findAll();
}).then(users => {
  console.log(users) // ... 以获取 user 对象的数组
})
```

一次更新几行:

```js
Task.bulkCreate([
  {subject: 'programming', status: 'executing'},
  {subject: 'reading', status: 'executing'},
  {subject: 'programming', status: 'finished'}
]).then(() => {
  return Task.update(
    { status: 'inactive' }, /* 设置属性的值 */,
    { where: { subject: 'programming' }} /* where 规则 */
  );
}).spread((affectedCount, affectedRows) => {
  // .update 在数组中返回两个值，因此我们使用 .spread
  // 请注意，affectedRows 只支持以 returning: true 的方式进行定义
  
  // affectedCount 将会是 2
  return Task.findAll();
}).then(tasks => {
  console.log(tasks) // “programming” 任务都将处于 “inactive” 状态
})
```

然后删除它们:

```js
Task.bulkCreate([
  {subject: 'programming', status: 'executing'},
  {subject: 'reading', status: 'executing'},
  {subject: 'programming', status: 'finished'}
]).then(() => {
  return Task.destroy({
    where: {
      subject: 'programming'
    },
    truncate: true /* 这将忽 where 并用 truncate table 替代  */
  });
}).then(affectedRows => {
  // affectedRows 将会是 2
  return Task.findAll();
}).then(tasks => {
  console.log(tasks) // 显示 tasks 内容
})
```

如果您直接从 user 接受值，则限制要实际插入的列可能会更好。`bulkCreate()` 接受一个选项对象作为第二个参数。 该对象可以有一个 `fields` 参数（一个数组），让它知道你想要明确构建哪些字段

```js
User.bulkCreate([
  { username: 'foo' },
  { username: 'bar', admin: true}
], { fields: ['username'] }).then(() => {
  // admin 将不会被构建
})
```

`bulkCreate`  最初是成为 主流/快速 插入记录的方法，但是有时您希望能够同时插入多行而不牺牲模型验证，即使您明确地告诉 Sequelize 去筛选哪些列。 你可以通过在options对象中添加一个 `validate: true` 属性来实现。

```js
const Tasks = sequelize.define('task', {
  name: {
    type: Sequelize.STRING,
    validate: {
      notNull: { args: true, msg: 'name cannot be null' }
    }
  },
  code: {
    type: Sequelize.STRING,
    validate: {
      len: [3, 10]
    }
  }
})
 
Tasks.bulkCreate([
  {name: 'foo', code: '123'},
  {code: '1234'},
  {name: 'bar', code: '1'}
], { validate: true }).catch(errors => {

  /* console.log(errors) 看起来像这样:
  [
    { record:
    ...
    errors:
      { name: 'SequelizeValidationError',
        message: 'Validation error',
        errors: [Object] } },
    { record:
      ...
      errors:
        { name: 'SequelizeValidationError',
        message: 'Validation error',
        errors: [Object] } }
  ]
  */
  
})
```

## 一个实例的值

如果你记录一个实例，你会注意到有很多额外的东西。 为了隐藏这些东西并将其减少到非常有趣的信息，您可以使用 `get` 属性。 使用选项 `plain: true` 调用它将只返回一个实例的值。

```js
Person.create({
  name: 'Rambow',
  firstname: 'John'
}).then(john => {
  console.log(john.get({
    plain: true
  }))
})
 
// 结果:
 
// { name: 'Rambow',
//   firstname: 'John',
//   id: 1,
//   createdAt: Tue, 01 May 2012 19:12:16 GMT,
//   updatedAt: Tue, 01 May 2012 19:12:16 GMT
// }
```

**提示:** 您还可以使用  `JSON.stringify(instance)` 将一个实例转换为 JSON。 基本上与  `values` 返回的相同。

## 重载实例

如果你需要让你的实例同步，你可以使用 `reload` 方法。 它将从数据库中获取当前数据，并覆盖调用该方法的模型的属性。

```js
Person.findOne({ where: { name: 'john' } }).then(person => {
  person.name = 'jane'
  console.log(person.name) // 'jane'
 
  person.reload().then(() => {
    console.log(person.name) // 'john'
  })
})
```

## 递增

为了增加实例的值而不发生并发问题，您可以使用 `increment`。

首先，你可以定义一个字段和要添加的值。

```js
User.findById(1).then(user => {
  return user.increment('my-integer-field', {by: 2})
}).then(user => {
  // Postgres默认会返回更新的 user (除非通过设置禁用 { returning: false })
  // 在其他方言中，您将需要调用 user.reload() 来获取更新的实例...
})
```

然后，你可以定义多个字段和要添加到其中的值。

```js
User.findById(1).then(user => {
  return user.increment([ 'my-integer-field', 'my-very-other-field' ], {by: 2})
}).then(/* ... */)
```

最后，你可以定义一个包含字段及其递增值的对象。

```js
User.findById(1).then(user => {
  return user.increment({
    'my-integer-field':    2,
    'my-very-other-field': 3
  })
}).then(/* ... */)
```

## 递减

为了减少一个实例的值而不遇到并发问题，你可以使用 `decrement`。

首先，你可以定义一个字段和要添加的值。

```js
User.findById(1).then(user => {
  return user.decrement('my-integer-field', {by: 2})
}).then(user => {
  // Postgres默认会返回更新的 user (除非通过设置禁用 { returning: false })
  // 在其他方言中，您将需要调用 user.reload() 来获取更新的实例...
})
```

然后，你可以定义多个字段和要添加到其中的值。

```js
User.findById(1).then(user => {
  return user.decrement([ 'my-integer-field', 'my-very-other-field' ], {by: 2})
}).then(/* ... */)
```

最后， 你可以定义一个包含字段及其递减值的对象。

```js
User.findById(1).then(user => {
  return user.decrement({
    'my-integer-field':    2,
    'my-very-other-field': 3
  })
}).then(/* ... */)
```
