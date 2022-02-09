# Hooks - 钩子

Hooks(也称为生命周期事件)是在执行 sequelize 中的调用之前和之后调用的函数. 例如,如果要在保存之前始终在模型上设置一个值,则可以添加一个 `beforeUpdate` hook.

**注意:** _你不能对实例使用 hook. Hook 用于模型._

## 可用的 hooks

Sequelize 提供了很多 hook. 完整列表可以直接在[源代码 - lib/hooks.js](https://github.com/sequelize/sequelize/blob/v6/lib/hooks.js#L7) 中找到.

## Hooks 触发顺序

下面显示了最常见的 hook 的触发顺序.

_**注意:** 此列表并不详尽._

```text
(1)
  beforeBulkCreate(instances, options)
  beforeBulkDestroy(options)
  beforeBulkUpdate(options)
(2)
  beforeValidate(instance, options)

[... validation happens ...]

(3)
  afterValidate(instance, options)
  validationFailed(instance, options, error)
(4)
  beforeCreate(instance, options)
  beforeDestroy(instance, options)
  beforeUpdate(instance, options)
  beforeSave(instance, options)
  beforeUpsert(values, options)

[... creation/update/destruction happens ...]

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

## 声明 Hooks

Hook 的参数通过引用传递. 这意味着你可以更改值,这将反映在 insert / update 语句中. 一个 hook 可能包含异步动作 - 在这种情况下,hook 函数应该返回一个 Promise.

当前有三种方法以编程方式添加 hook：

```js
// 方法 1 通过 .init() 方法
class User extends Model {}
User.init({
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
  },
  sequelize
});

// 方法 2 通过 .addHook() 方法
User.addHook('beforeValidate', (user, options) => {
  user.mood = 'happy';
});

User.addHook('afterValidate', 'someCustomName', (user, options) => {
  return Promise.reject(new Error("I'm afraid I can't let you do that!"));
});

// 方法 3 通过 direct 方法
User.beforeCreate(async (user, options) => {
  const hashedPassword = await hashPassword(user.password);
  user.password = hashedPassword;
});

User.afterValidate('myHookAfter', (user, options) => {
  user.username = 'Toni';
});
```

## 删除 hooks

只能删除带有名称参数的 hook.

```js
class Book extends Model {}
Book.init({
  title: DataTypes.STRING
}, { sequelize });

Book.addHook('afterCreate', 'notifyUsers', (book, options) => {
  // ...
});

Book.removeHook('afterCreate', 'notifyUsers');
```

你可以有许多同名的 hook. 调用 `.removeHook()` 将删除所有对象.

## 全局 / 通用 hooks

全局 hook 是所有模型运行的 hook. 它们对于插件特别有用, 并且可以为所有模型定义您想要的行为. 例如允许在您的模型上使用 `sequelize.define` 自定义时间戳:

```js
const User = sequelize.define('User', {}, {
    tableName: 'users',
    hooks : {
        beforeCreate : (record, options) => {
            record.dataValues.createdAt = new Date().toISOString().replace(/T/, ' ').replace(/\..+/g, '');
            record.dataValues.updatedAt = new Date().toISOString().replace(/T/, ' ').replace(/\..+/g, '');
        },
        beforeUpdate : (record, options) => {
            record.dataValues.updatedAt = new Date().toISOString().replace(/T/, ' ').replace(/\..+/g, '');
        }
    }
});
```

它们可以通过多种方式定义, 语义略有不同:

### 默认 Hooks (在 Sequelize 构造函数参数)

```js
const sequelize = new Sequelize(..., {
  define: {
    hooks: {
      beforeCreate() {
        // 做点什么
      }
    }
  }
});
```

这会向所有模型添加一个默认 hook,如果模型未定义自己的 `beforeCreate` hook,则将运行该 hook：

```js
const User = sequelize.define('User', {});
const Project = sequelize.define('Project', {}, {
  hooks: {
    beforeCreate() {
      // 做点其他事
    }
  }
});

await User.create({});    // 运行全局 hook
await Project.create({}); // 运行自己的 hook (因为全局 hook 被覆盖)
```

### 常驻 Hooks (通过 `sequelize.addHook`)

```js
sequelize.addHook('beforeCreate', () => {
  // 做点什么
});
```

无论模型是否指定自己的 `beforeCreate` hook,该 hook 始终运行. 本地 hook 总是在全局 hook 之前运行：

```js
const User = sequelize.define('User', {});
const Project = sequelize.define('Project', {}, {
  hooks: {
    beforeCreate() {
      // 做点其他事
    }
  }
});

await User.create({});    // 运行全局 hook
await Project.create({}); // 运行自己的 hook, 其次是全局 hook
```

也可以在传递给 Sequelize 构造函数的参数中定义常驻 hook：

```js
new Sequelize(..., {
  hooks: {
    beforeCreate() {
      // 做点什么
    }
  }
});
```

请注意,以上内容与上述 *默认 Hooks* 不同. 那就是使用构造函数的 `define` 参数. 这里不是.

### 连接 Hooks

Sequelize 提供了四个 hook,它们在获得或释放数据库连接之前和之后立即执行：

* `sequelize.beforeConnect(callback)`
  * 回调具有以下形式 `async (config) => /* ... */`
* `sequelize.afterConnect(callback)`
  * 回调具有以下形式 `async (connection, config) => /* ... */`
* `sequelize.beforeDisconnect(callback)`
  * 回调具有以下形式 `async (connection) => /* ... */`
* `sequelize.afterDisconnect(callback)`
  * 回调具有以下形式 `async (connection) => /* ... */`

如果你需要异步获取数据库凭据,或者需要在创建低级数据库连接后直接访问它,这些 hook 很有用.

例如,我们可以从令牌存储异步获取数据库密码,并使用新的凭证对 Sequelize 的配置对象进行更新：

```js
sequelize.beforeConnect(async (config) => {
  config.password = await getAuthToken();
});
```

这些 hook 只能 *被* 声明为永久全局 hook,因为所有模型都共享连接池.

## 实例 hooks

每当你编辑单个对象时,都会触发以下 hook：

* `beforeValidate`
* `afterValidate` / `validationFailed`
* `beforeCreate` / `beforeUpdate` / `beforeSave` / `beforeDestroy`
* `afterCreate` / `afterUpdate` / `afterSave` / `afterDestroy`

```js
User.beforeCreate(user => {
  if (user.accessLevel > 10 && user.username !== "Boss") {
    throw new Error("You can't grant this user an access level above 10!");
  }
});
```

以下示例将引发错误:

```js
try {
  await User.create({ username: 'Not a Boss', accessLevel: 20 });
} catch (error) {
  console.log(error); // 你不能授予该用户10以上的访问权限！
};
```

以下示例将成功:

```js
const user = await User.create({ username: 'Boss', accessLevel: 20 });
console.log(user); // 用户对象,用户名 `Boss`,accessLevel 为 20
```

### 模型 hooks

有时你会使用诸如 `bulkCreate`, `update` 和 `destroy` 之类的方法一次编辑多个记录. 每当你使用这些方法之一时,就会触发以下 hook：

* `YourModel.beforeBulkCreate(callback)`
  * 回调具有以下形式 `(instances, options) => /* ... */`
* `YourModel.beforeBulkUpdate(callback)`
  * 回调具有以下形式 `(options) => /* ... */`
* `YourModel.beforeBulkDestroy(callback)`
  * 回调具有以下形式 `(options) => /* ... */`
* `YourModel.afterBulkCreate(callback)`
  * 回调具有以下形式 `(instances, options) => /* ... */`
* `YourModel.afterBulkUpdate(callback)`
  * 回调具有以下形式 `(options) => /* ... */`
* `YourModel.afterBulkDestroy(callback)`
  * 回调具有以下形式 `(options) => /* ... */`

注意：默认情况下,类似 `bulkCreate` 的方法不会触发单独的 hook - 仅批量 hook. 但是,如果你还希望触发单个 hook,则可以将 `{ individualHooks: true }` 参数传递给查询调用. 但是,这可能会严重影响性能,具体取决于所涉及的记录数(因为,除其他外,所有实例都将被加载到内存中). 例子：

```js
await Model.destroy({
  where: { accessLevel: 0 },
  individualHooks: true
});
// 这将选择所有将要删除的记录,并在每个实例上触发 `beforeDestroy` 和 `afterDestroy`.

await Model.update({ username: 'Tony' }, {
  where: { accessLevel: 0 },
  individualHooks: true
});
// 这将选择所有将要更新的记录,并在每个实例上发出 `beforeUpdate` 和 `afterUpdate`.
```

如果你将 `Model.bulkCreate(...)` 与 `updateOnDuplicate` 参数一起使用,则对 hook 中对 `updateOnDuplicate` 数组中未提供的字段所做的更改将不会保留到数据库中. 但是,如果需要的话,可以在挂钩中更改 `updateOnDuplicate` 参数.

```js
User.beforeBulkCreate((users, options) => {
  for (const user of users) {
    if (user.isMember) {
      user.memberSince = new Date();
    }
  }

  // 将 `memberSince` 添加到 updateOnDuplicate,否则它将不会持久化
  if (options.updateOnDuplicate && !options.updateOnDuplicate.includes('memberSince')) {
    options.updateOnDuplicate.push('memberSince');
  }
});

// 使用 updateOnDuplicate 参数批量更新现有用户
await Users.bulkCreate([
  { id: 1, isMember: true },
  { id: 2, isMember: false }
], {
  updateOnDuplicate: ['isMember']
});
```

## 关联

在大多数情况下,hook 在关联时对实例的作用相同.

### 一对一和一对多关联

* 当使用 `add`/`set` mixin 方法时,`beforeUpdate` 和 `afterUpdate` hook 将运行.

* `beforeDestroy` 和 `afterDestroy` hook 只会在具有 `onDelete: 'CASCADE'` 和 `hooks: true` 的关联上被调用. 例如

```js
class Projects extends Model {}
Projects.init({
  title: DataTypes.STRING
}, { sequelize });

class Tasks extends Model {}
Tasks.init({
  title: DataTypes.STRING
}, { sequelize });

Projects.hasMany(Tasks, { onDelete: 'CASCADE', hooks: true });
Tasks.belongsTo(Projects);
```

该代码将在 Tasks 模型上运行 `beforeDestroy` 和 `afterDestroy` hook.

默认情况下,Sequelize 将尝试尽可能优化你的查询. 当在删除时调用级联时,Sequelize 将简单地执行：

```sql
DELETE FROM `table` WHERE associatedIdentifier = associatedIdentifier.primaryKey
```

但是,添加 `hooks: true` 会明确告诉 Sequelize 优化与你无关. 然后,Sequelize 首先将对关联的对象执行 `SELECT` 并逐个销毁每个实例,以便能够正确调用 hook(使用正确的参数).

### 多对多关联

* 当对 `belongsToMany` 关系使用 `add` mixin 方法时(将一个或多个记录添加到联结表中),联结模型中的 `beforeBulkCreate` 和 `afterBulkCreate` hook 将运行.
  * 如果将 `{ individualHooks: true }` 传递给该调用,则每个单独的 hook 也将运行.

* 当对 `belongsToMany` 关系使用 `remove` mixin 方法时(将一个或多个记录删除到联结表中),联结模型中的 `beforeBulkDestroy` 和 `afterBulkDestroy` hook 将运行.
  * 如果将 `{ individualHooks: true }` 传递给该调用,则每个单独的 hook 也将运行.

如果你的关联是多对多,则在使用 `remove` 调用时,你可能会对在直通模型上触发 hook 感兴趣. 在内部,sequelize 使用的是 `Model.destroy`,从而导致在每个直通实例上调用 `bulkDestroy` 而不是 `before/afterDestroy` hook.

## Hooks 和 事务

Sequelize 中的许多模型操作都允许你在方法的 options 参数中指定事务. 如果在原始调用中指定了事务,则该事务将出现在传递给 hook 函数的 options 参数中. 例如,考虑以下代码片段：

```js
User.addHook('afterCreate', async (user, options) => {
  // 我们可以使用 `options.transaction` 来执行
  // 与触发此 hook 的调用相同的事务来执行其他一些调用
  await User.update({ mood: 'sad' }, {
    where: {
      id: user.id
    },
    transaction: options.transaction
  });
});

await sequelize.transaction(async t => {
  await User.create({
    username: 'someguy',
    mood: 'happy'
  }, {
    transaction: t
  });
});
```

如果在前面的代码中对 `User.update` 的调用中未包含 transaction 参数,则不会发生更改,因为在提交未决事务之前,数据库中不存在我们新创建的用户.

### 内部事务

重要的是要认识到,sequelize 可能会在内部对某些操作(例如`Model.findOrCreate`)使用事务. 如果 hook 函数执行依赖于数据库中对象存在的读取或写入操作,或者像上一节中的示例一样修改对象的存储值,则应始终指定`{transaction：options.transaction}`：

* 如果使用了事务,则 `{transaction：options.transaction}` 将确保再次使用该事务;
* 否则,`{ transaction: options.transaction }` 等同于 `{ transaction: undefined }`,这将不使用事务.

这样,你的 hook 将始终正确运行.