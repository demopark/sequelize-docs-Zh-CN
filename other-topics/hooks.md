# Hooks - 钩子

Hook 是你可以监听的事件, 当调用相应的方法时会触发这些事件. 

它们对于向应用程序的核心添加自定义功能很有用. 
例如, 如果你想在保存模型之前始终在模型上设置一个值, 你可以监听模型的 `beforeUpdate` hook. 

## 静态 Sequelize Hook

`Sequelize` 类支持以下 hook:

| Hook name                  | Async | Run when                        |
|----------------------------|-------|---------------------------------|
|  `beforeInit`, `afterInit` | ❌     | 创建一个 sequelize 实例 |

示例:

```typescript
import { Sequelize } from '@sequelize/core';

Sequelize.hooks.addListener('beforeInit', () => {
  console.log('A new sequelize instance is being created');
});
```

## 实例 Sequelize Hook

`Sequelize` 实例支持以下 hook：

| Hook name                             | Async | Run when                                                   |
|---------------------------------------|-------|------------------------------------------------------------|
| `beforeDefine`, `afterDefine`         | ❌     | 正在注册一个新的 `Model` 类                    |
| `beforeQuery`, `afterQuery`           | ✅     | 正在运行 SQL 查询                                   |
| `beforeBulkSync`, `afterBulkSync`     | ✅     | `sequelize.sync` 被调用                                 |
| `beforeConnect`, `afterConnect`       | ✅     | 每当创建与数据库的新连接时 |
| `beforeDisconnect`, `afterDisconnect` | ✅     | 每当关闭与数据库的连接时      |

**注意**

所有 [模型 hook] 也会在 sequelize 实例上触发：

```typescript
import { Sequelize } from '@sequelize/core';

const sequelize = new Sequelize(/* options */);

// 每当在任何模型上调用 findAll 时都会调用它
sequelize.hooks.addListener('beforeFind', () => {
  console.log('findAll 已被模型调用');
});
```

### 注册 Sequelize Hook

实例 Sequelize hook 可以通过两种方式注册:

1. 使用 `Sequelize` 实例的 `hooks` 属性:

   ```typescript
   import { Sequelize } from '@sequelize/core';
  
   const sequelize = new Sequelize(/* options */);
  
   // highlight-next-line
   sequelize.hooks.addListener('beforeDefine', () => {
     console.log('正在初始化一个新模型');
   });
   ```
   
2. 通过 `Sequelize` 参数:

   ```typescript
   import { Sequelize } from '@sequelize/core';
  
   const sequelize = new Sequelize({
     // highlight-next-line
     hooks: {
       beforeDefine: () => {
         console.log('正在初始化一个新模型');
       },
     },
   });
   ```

## 模型 Sequelize Hook

`Model` 类支持以下 hook：

| Hook name                                                                              | Async | Run when                                                                        |
|----------------------------------------------------------------------------------------|-------|---------------------------------------------------------------------------------|
| `beforeAssociate`, `afterAssociate`                                                    | ❌     | 每当在模型上声明关联时                               |
| `beforeSync`, `afterSync`                                                              | ✅     | 当调用 `sequelize.sync` 或 `Model.sync` 时                                |
| `beforeValidate`, `afterValidate`, `validationFailed`                                  | ✅     | 当验证模型的属性时(发生在大多数模型方法中) |
| `beforeFind`, `beforeFindAfterExpandIncludeAll`, `beforeFindAfterOptions`, `afterFind` | ✅     | 当调用 `Model.findAll` 时[^find-all]                                       |
| `beforeCount`                                                                          | ✅     | 当调用 `Model.count` 时                                                    |
| `beforeUpsert`, `afterUpsert`                                                          | ✅     | 当调用 `Model.upsert` 时                                                   |
| `beforeBulkCreate`, `afterBulkCreate`                                                  | ✅     | 当调用 `Model.bulkCreate` 时                                               |
| `beforeBulkDestroy`, `afterBulkDestroy`                                                | ✅     | 当调用 `Model.destroy` 时                                                  |
| `beforeDestroy`, `afterDestroy`                                                        | ✅     | 当调用 `Model#destroy` 时                                                  |
| `beforeBulkRestore`, `afterBulkRestore`                                                | ✅     | 当调用 `Model.restore` 时                                                  |
| `beforeRestore`, `afterRestore`                                                        | ✅     | 当调用 `Model#restore` 时                                                  |
| `beforeBulkUpdate`, `afterBulkUpdate`                                                  | ✅     | 当调用 `Model.update` 时                                                   |
| `beforeUpdate`, `afterUpdate`                                                          | ✅     | 当调用 `Model#update` 或 `Model#save` 时(并且模型不是新的)         |
| `beforeCreate`, `afterCreate`                                                          | ✅     | 当调用 `Model#save` 时(并且模型是新的)                              |
| `beforeSave`, `afterSave`                                                              | ✅     | 当调用 `Model#update` 或 `Model#save` 时                                   |

### 注册模型 hook

实例 Sequelize hook 可以通过三种方式注册:

1. 使用 `Sequelize` 实例的 `hooks` 属性:

   ```typescript
   import { Sequelize, DataTypes } from '@sequelize/core';
    
   const sequelize = new Sequelize(/* options */);
    
   const MyModel = sequelize.define('MyModel', {
     name: DataTypes.STRING,
   });
   
   // highlight-next-line
   MyModel.hooks.addListener('beforeFind', () => {
     console.log('findAll has been called on MyModel');
   });
   ```
2. 通过 `Model.init` 或 `sequelize.define` 的参数:

   ```typescript
   import { DataTypes } from '@sequelize/core';

   const MyModel = sequelize.define('MyModel', {
     name: DataTypes.STRING,
   }, {
     // highlight-next-line
     hooks: {
       beforeFind: () => {
         console.log('findAll has been called on MyModel');
       },
     },
   });
   ```
3. 使用装饰器

   ```typescript
   import { Sequelize, Model, Hook } from '@sequelize/core';
   import { BeforeFind } from '@sequelize/core/decorators-legacy';
   
   export class MyModel extends Model {
     // highlight-next-line
     @BeforeFind
     static logFindAll() {
       console.log('findAll has been called on MyModel');
     }
   }
   ```

## 同步 vs 异步 Hook

在上表中, 你可以看到一些挂钩被标记为 `Async`.
这意味着你的侦听器可以返回一个将在继续执行 hook 之前等待的 `Promise`.

在 hook 中进行异步操作时要小心.  hook 是连续运行的, 在你的 hook 被解析之前不会运行下一个 hook. 
如果你有很多 hook, 这可能会导致性能问题. 

尝试从同步 hook 返回一个 `Promise` 会引发错误. 

## 移除 Hook

你可以使用 `hooks.removeListener` 并提供与 `hooks.addListener` 相同的回调或你的侦听器名称来移除 hook：

```js
class Book extends Model {}
Book.init({
  title: DataTypes.STRING
}, { sequelize });

const myListener = (book, options) => {
  // ...
};

Book.hooks.addListener('afterCreate', 'yourHookIdentifier', myListener);

// 这两种都将删除 hook:
// success-next-line
Book.hooks.removeListener('afterCreate', myListener);
// success-next-line
Book.hooks.removeListener('afterCreate', 'yourHookIdentifier');
```

**注意**

装饰器添加的 hook 不能使用回调实例删除, 但如果它们被命名仍然可以删除:

```typescript
import { Sequelize, Model, Hook } from '@sequelize/core';
import { BeforeFind } from '@sequelize/core/decorators-legacy';

export class MyModel extends Model {
  @BeforeFind({ name: 'yourHookIdentifier' })
  static logFindAll() {
    console.log('findAll has been called on MyModel');
  }
}

// 这行不通
// error-next-line
MyModel.hooks.removeListener('beforeFind', MyModel.logFindAll);

// 但是这种可以
// success-next-line
MyModel.hooks.removeListener('beforeFind', 'yourHookIdentifier');
```

但是, 我们不建议删除通过装饰器添加的 hook, 因为这可能会使你的代码更难理解. 


## 关联方法 Hook

[关联](../core-concepts/assocs.md) 在你的模型上添加的方法确实提供了特定于它们的 hook, 但它们是建立在顶部的
常规模型方法, 这将触发 hook. 

例如, 使用 `add` / `set` 混合方法将触发 `beforeUpdate` 和 `afterUpdate` hook. 

## `individualHooks`

**注意**

不鼓励使用此参数有两个原因:

- 与 "普通" hook 不同, 无法修改提供给 `individualHooks` 发出的事件的数据.  只能修改 "批量" 事件的日期. 
- 这个参数很慢, 因为它要获取需要销毁的实例.

小心使用.  如果你需要对数据库更改做出应对, 请考虑改用数据库的触发器和通知系统. 

当使用 `Model.destroy` 或 `Model.update` 等静态模型方法时, 只会调用它们对应的 "批量" hook(例如 `beforeBulkDestroy`), 而不是实例 hook (例如 `beforeDestroy`). 

如果要触发实例 hook, 请使用方法的 `individualHooks` 参数为将受到影响的每一行运行实例 hook.

以下示例将为将要删除的每一行触发 `beforeDestroy` 和 `afterDestroy` hook:

```js
User.destroy({
  where: {
    id: [1, 2, 3],
  },
  individualHooks: true,
});
```

## 例外情况

只有 __Model 方法__ 触发hook.  这意味着在许多情况下, Sequelize 将与数据库交互而不触发 hook. 

这些包括但不限于:

- 由于 `ON DELETE CASCADE` 约束而被数据库删除的实例, [除非 `hooks` 参数为 true].
- 由于 `SET NULL` 或 `SET DEFAULT` 约束, 数据库正在更新实例.
- [Raw 查询](../core-concepts/raw-queries.md).
- 所有 QueryInterface 方法.

如果你需要对这些事件做出应对, 请考虑改用数据库的本机和通知系统. 

## 级联删除的 Hook

如 [例外情况] 中所示, 由于 `ON DELETE CASCADE` 约束, 当数据库删除实例时, Sequelize 不会触发挂钩. 

但是, 如果你在定义关联时将 `hooks` 参数设置为 `true`, Sequelize 将为已删除的实例触发 `beforeDestroy` 和 `afterDestroy` hook. 

**注意**

由于以下原因, 不鼓励使用此参数:

- 这个参数需要很多额外的查询.  `destroy` 方法通常执行单个查询. 如果启用此参数, 将执行一个额外的 `SELECT` 查询, 以及一个额外的 `DELETE` 查询, 用于选择返回的每一行.
- 如果你不在事务中运行此查询, 并且发生错误, 你最终可能会删除一部分行, 而一些行没有被删除. 
- 此参数仅在使用 `destroy` 的 *实例* 版本时有效.  静态版本不会触发 hook, 即使有 `individualHooks`.
- 此参数在 `paranoid` 模式下不起作用.
- 如果你只在拥有外键的模型上定义关联, 则此参数将不起作用.  你还需要定义反向关联.

此参数被视为遗留选项.  如果你需要收到数据库更改通知, 我们强烈建议你使用数据库的触发器和通知系统. 

以下是如何使用此参数的示例:

```ts
import { Model } from '@sequelize/core';
import { HasMany, BeforeDestroy } from '@sequelize/core/decorators-legacy';

class User extends Model {
  // 这个"hook"参数将导致"beforeDestroy"和"afterDestroy"
  // highlight-next-line
  @HasMany(() => Post, { hooks: true })
  declare posts: Post[];
}

class Post extends Model {
  @BeforeDestroy
  static logDestroy() {
    console.log('帖子已被销毁');
  }
}

const sequelize = new Sequelize({
  /* 参数 */
  models: [User, Post], 
});

await sequelize.sync({ force: true });

const user = await User.create();
const post = await Post.create({ userId: user.id });

// 这会输出 "帖子已被销毁"
await user.destroy();
```

## Hook 和 事务

Sequelize 中的许多模型操作都支持在方法的参数中指定事务. 如果在原始调用中 *指定* 了事务, 它将出现在传递给 hook 函数的参数中. 

例如, 考虑以下代码片段:

```js
User.hooks.addListener('afterCreate', async (user, options) => {
  // 我们可以使用 `options.transaction` 
  // 使用与触发此 hook 的调用相同的事务来执行其他调用
  await User.update({ mood: 'sad' }, {
    where: {
      id: user.id
    },
    // highlight-next-line
    transaction: options.transaction
  });
});

await sequelize.transaction(async transaction => {
  await User.create({
    username: 'someguy',
    mood: 'happy'
  }, {
    transaction,
  });
});
```

如果我们在前面的代码中调用 User.update 时没有包含事务参数, 则不会发生任何变化, 因为我们新创建的用户在待处理事务之前不存在于数据库中. 

[^find-all]: **findAll**: 请注意, 一些方法, 例如 `Model.findOne`、`Model.findAndCountAll` 和关联获取器也会在内部调用 `Model.findAll`.  这也会导致为这些方法调用 `beforeFind` hook. 