# Hooks - 钩子

Hook（也称为生命周期事件）是执行 sequelize 调用之前和之后调用的函数。 例如，如果要在保存模型之前始终设置值，可以添加一个 `beforeUpdate`  hook。

获取完整列表, 请查看 [Hooks file](https://github.com/sequelize/sequelize/blob/master/lib/hooks.js#L7).

## 操作清单

```
(1)
  beforeBulkCreate(instances, options)
  beforeBulkDestroy(options)
  beforeBulkUpdate(options)
(2)
  beforeValidate(instance, options)
(-)
  validate
(3)
  afterValidate(instance, options)
  - or -
  validationFailed(instance, options, error)
(4)
  beforeCreate(instance, options)
  beforeDestroy(instance, options)
  beforeUpdate(instance, options)
  beforeSave(instance, options)
  beforeUpsert(values, options)
(-)
  create
  destroy
  update
(5)
  afterCreate(instance, options)
  afterDestroy(instance, options)
  afterUpdate(instance, options)
  afterSave(instance, options)
  afterUpsert(created, options)
(6)
  afterBulkCreate(instances, options)
  afterBulkDestroy(options)
  afterBulkUpdate(options)
```

## 声明 Hook

Hook 的参数通过引用传递。 这意味着您可以更改值，这将反映在insert / update语句中。 Hook 可能包含异步动作 - 在这种情况下，Hook 函数应该返回一个 promise。

目前有三种以编程方式添加 hook 的方法:

```js
// 方法1 通过 .define() 方法
const User = sequelize.define('user', {
  username: DataTypes.STRING,
  mood: {
    type: DataTypes.ENUM,
    values: ['happy', 'sad', 'neutral']
  }
}, {
  hooks: {
    beforeValidate: (user, options) => {
      user.mood = 'happy';
    },
    afterValidate: (user, options) => {
      user.username = 'Toni';
    }
  }
});

// 方法2 通过 . hook() 方法 (或其别名 .addHook() 方法)
User.hook('beforeValidate', (user, options) => {
  user.mood = 'happy';
});

User.addHook('afterValidate', 'someCustomName', (user, options) => {
  return sequelize.Promise.reject(new Error("I'm afraid I can't let you do that!"));
});

// 方法3 通过直接方法
User.beforeCreate((user, options) => {
  return hashPassword(user.password).then(hashedPw => {
    user.password = hashedPw;
  });
});

User.afterValidate('myHookAfter', (user, options) => {
  user.username = 'Toni';
});
```

## 移除 Hook

只能删除有名称参数的 hook。

```js
const Book = sequelize.define('book', {
  title: DataTypes.STRING
});

Book.addHook('afterCreate', 'notifyUsers', (book, options) => {
  // ...
});

Book.removeHook('afterCreate', 'notifyUsers');
```

你可以有很多同名的 hook。 调用 `.removeHook()` 将会删除它们。

## 全局 / 通用 Hook

全局 hook 是所有模型的 hook。 他们可以定义您想要的所有模型的行为，并且对插件特别有用。 它们可以用两种方式来定义，它们的语义略有不同：

### Sequelize.options.define (默认 hook)
```js
const sequelize = new Sequelize(..., {
    define: {
        hooks: {
            beforeCreate: () => {
                // 做些什么
            }
        }
    }
});
```

这将为所有模型添加一个默认 hook，如果模型没有定义自己的 `beforeCreate`  hook，那么它将运行。

```js
const User = sequelize.define('user');
const Project = sequelize.define('project', {}, {
    hooks: {
        beforeCreate: () => {
            //  做些其它什么
        }
    }
});

User.create() // 运行全局 hook
Project.create() // 运行其自身的 hook (因为全局 hook 被覆盖)
```

### Sequelize.addHook (常驻 hook)

```js
sequelize.addHook('beforeCreate', () => {
    // 做些什么
});
```

这个 hook 总是在创建之前运行，无论模型是否指定了自己的 `beforeCreate`  hook：


```js
const User = sequelize.define('user');
const Project = sequelize.define('project', {}, {
    hooks: {
        beforeCreate: () => {
            // 做些其它什么
        }
    }
});

User.create() // 运行全局 hook
Project.create() //运行其自己的 hook 之后运行全局 hook
```

本地 hook 总是在全局 hook 之前运行。


### 实例 Hook

当您编辑单个对象时，以下 hook 将触发

```
beforeValidate
afterValidate or validationFailed
beforeCreate / beforeUpdate  / beforeDestroy
afterCreate / afterUpdate / afterDestroy
```

```js
// ...定义 ...
User.beforeCreate(user => {
  if (user.accessLevel > 10 && user.username !== "Boss") {
    throw new Error("您不能授予该用户10级以上的访问级别！")
  }
})
```

此示例将返回错误:

```js
User.create({username: 'Not a Boss', accessLevel: 20}).catch(err => {
  console.log(err); // 您不能授予该用户 10 级以上的访问级别！
});
```

以下示例将返回成功:

```js
User.create({username: 'Boss', accessLevel: 20}).then(user => {
  console.log(user); // 用户名为 Boss 和 accessLevel 为 20 的用户对象
});
```

### 模型 Hook

有时，您将一次编辑多个记录，方法是使用模型上的 `bulkCreate, update, destroy` 方法。 当您使用以下方法之一时，将会触发以下内容：

```
beforeBulkCreate(instances, options)
beforeBulkUpdate(options)
beforeBulkDestroy(options)
afterBulkCreate(instances, options)
afterBulkUpdate(options)
afterBulkDestroy(options)
```

如果要为每个单独的记录触发 hook，连同批量 hook，您可以将 `personalHooks:true` 传递给调用。

```js
Model.destroy({ where: {accessLevel: 0}, individualHooks: true});
// 将选择要删除的所有记录，并在每个实例删除之前 + 之后触发

Model.update({username: 'Toni'}, { where: {accessLevel: 0}, individualHooks: true});
// 将选择要更新的所有记录，并在每个实例更新之前 + 之后触发
```

Hook 方法的 `options` 参数将是提供给相应方法或其克隆和扩展版本的第二个参数。

```js
Model.beforeBulkCreate((records, {fields}) => {
  // records = 第一个参数发送到 .bulkCreate
  // fields = 第二个参数字段之一发送到 .bulkCreate
  })

Model.bulkCreate([
    {username: 'Toni'}, // 部分记录参数
    {username: 'Tobi'} // 部分记录参数
  ], {fields: ['username']} // 选项参数
)

Model.beforeBulkUpdate(({attributes, where}) => {
  // where - 第二个参数的克隆的字段之一发送到 .update
  // attributes - .update 的第二个参数的克隆的字段之一被用于扩展
})

Model.update({gender: 'Male'} /*属性参数*/, { where: {username: 'Tom'}} /*where 参数*/)

Model.beforeBulkDestroy(({where, individualHooks}) => {
  // individualHooks - 第二个参数被扩展的克隆被覆盖的默认值发送到 Model.destroy
  // where - 第二个参数的克隆的字段之一发送到 Model.destroy
})

Model.destroy({ where: {username: 'Tom'}} /*where 参数*/)
```

如果用 `updates.OnDuplicate` 参数使用 `Model.bulkCreate(...)` ，那么 hook 中对 `updatesOnDuplicate` 数组中没有给出的字段所做的更改将不会被持久保留到数据库。 但是，如果这是您想要的，则可以更改 hook 中的 updatesOnDuplicate 选项。

```js
// 使用 updatesOnDuplicate 选项批量更新现有用户
Users.bulkCreate([
  { id: 1, isMember: true },
  { id: 2, isMember: false }
], {
  updatesOnDuplicate: ['isMember']
});

User.beforeBulkCreate((users, options) => {
  for (const user of users) {
    if (user.isMember) {
      user.memberSince = new Date();
    }
  }

  // 添加 memberSince 到 updatesOnDuplicate 否则 memberSince 期将不会被保存到数据库
  options.updatesOnDuplicate.push('memberSince');
});
```

## 关联

在大多数情况下，hook 对于相关联的实例而言将是一样的，除了几件事情之外。

1. 当使用 add/set 函数时，将运行 beforeUpdate/afterUpdate hook。
2. 调用 beforeDestroy/afterDestroy hook 的唯一方法是与 `onDelete:'cascade` 和参数 `hooks：true` 相关联。 例如：

```js
const Projects = sequelize.define('projects', {
  title: DataTypes.STRING
});

const Tasks = sequelize.define('tasks', {
  title: DataTypes.STRING
});

Projects.hasMany(Tasks, { onDelete: 'cascade', hooks: true });
Tasks.belongsTo(Projects);
```

该代码将在Tasks表上运行beforeDestroy / afterDestroy。 默认情况下，Sequelize会尝试尽可能优化您的查询。 在删除时调用级联，Sequelize将简单地执行一个

```sql
DELETE FROM `table` WHERE associatedIdentifier = associatedIdentifier.primaryKey
```

然而，添加 `hooks: true` 会明确告诉 Sequelize，优化不是你所关心的，并且会在关联的对象上执行一个 `SELECT`，并逐个删除每个实例，以便能够使用正确的参数调用 hook。

如果您的关联类型为 `n:m`，则在使用 `remove` 调用时，您可能有兴趣在直通模型上触发 hook。 在内部，sequelize 使用 `Model.destroy`，致使在每个实例上调用 `bulkDestroy` 而不是 `before / afterDestroy`  hook。

这可以通过将 `{individualHooks:true}` 传递给 `remove` 调用来简单地解决，从而导致每个 hook 都通过实例对象被删除。


## 关于事务的注意事项

请注意，Sequelize 中的许多模型操作允许您在方法的 options 参数中指定事务。 如果在原始调用中 _指定_ 了一个事务，它将出现在传递给 hook 函数的 options 参数中。 例如，请参考以下代码段：

```js
// 这里我们使用异步 hook 的 promise 风格，而不是回调。
User.hook('afterCreate', (user, options) => {
  // 'transaction' 将在 options.transaction 中可用

  // 此操作将成为与原始 User.create 调用相同的事务的一部分。
  return User.update({
    mood: 'sad'
  }, {
    where: {
      id: user.id
    },
    transaction: options.transaction
  });
});


sequelize.transaction(transaction => {
  User.create({
    username: 'someguy',
    mood: 'happy',
    transaction
  });
});
```

如果我们在上述代码中的 `User.update` 调用中未包含事务选项，则不会发生任何更改，因为在已提交挂起的事务之前，我们新创建的用户不存在于数据库中。

### 内部事务

要认识到 sequelize 可能会在某些操作（如 `Model.findOrCreate`）内部使用事务是非常重要的。 如果你的 hook 函数执行依赖对象在数据库中存在的读取或写入操作，或者修改对象的存储值，就像上一节中的例子一样，你应该总是指定 `{ transaction: options.transaction }`。

如果在处理操作的过程中已经调用了该 hook ，则这将确保您的依赖读/写是同一事务的一部分。 如果 hook 没有被处理，你只需要指定`{ transaction: null }` 并且可以预期默认行为。