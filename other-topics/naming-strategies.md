# Naming Strategies - 命名策略

## `underscored` 参数

Sequelize 为模型提供了 `underscored` 参数. 设为 `true` 时,此参数会将所有属性的 `field` 参数设置为其名称的 [snake_case](https://en.wikipedia.org/wiki/Snake_case) 版本. 这也适用于由关联和其他自动生成的字段自动生成的外键. 例：

```js
const User = sequelize.define('task', { username: Sequelize.STRING }, {
  underscored: true
});
const Task = sequelize.define('task', { title: Sequelize.STRING }, {
  underscored: true
});
User.hasMany(Task);
Task.belongsTo(User);
```

上面我们有模型 User 和 Task,都使用了 `underscored` 的参数. 他们之间也有一对多的关系. 另外,回想一下,由于默认情况下 `timestamps` 为 `true`,因此我们应该期望 `createedAt` 和 `updatedAt` 字段也将自动创建.

如果没有 `underscored` 参数,Sequelize 会自动定义：

* 每个模型的 `createdAt` 属性,指向每个表中名为 `createdAt` 的列
* 每个模型的 `updatedAt` 属性,指向每个表中名为 `updatedAt` 的列
* `Task` 模型中的 `userId` 属性,指向任务表中名为 `userId` 的列

启用 `underscored` 参数后,Sequelize 将改为定义：

* 每个模型的 `createdAt` 属性,指向每个表中名为 `created_at ` 的列
* 每个模型的 `updatedAt` 属性,指向每个表中名为 `updated_at ` 的列
* `Task` 模型中的 `userId` 属性,指向任务表中名为 `user_id ` 的列

请注意,在这两种情况下,JavaScript 字段均仍为 [camelCase](https://en.wikipedia.org/wiki/Camel_case); 此参数仅更改这些字段如何映射到数据库本身. 每个属性的 `field` 参数都设置为它们的 snake_case 版本,但属性本身仍为 camelCase.

这样,在上面的代码上调用 `sync()` 将会生成以下内容：

```sql
CREATE TABLE IF NOT EXISTS "users" (
  "id" SERIAL,
  "username" VARCHAR(255),
  "created_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updated_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  PRIMARY KEY ("id")
);
CREATE TABLE IF NOT EXISTS "tasks" (
  "id" SERIAL,
  "title" VARCHAR(255),
  "created_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updated_at" TIMESTAMP WITH TIME ZONE NOT NULL,
  "user_id" INTEGER REFERENCES "users" ("id") ON DELETE SET NULL ON UPDATE CASCADE,
  PRIMARY KEY ("id")
);
```

## 单数与复数

乍看之下,在 Sequelize 中是否应使用名称的单数形式或复数形式可能会造成混淆. 本节旨在澄清这一点.

回想一下 Sequelize 在后台使用了一个名为 [inflection](https://www.npmjs.com/package/inflection) 的库,以便正确计算不规则的复数形式(例如 `person -> people`). 但是,如果你使用的是另一种语言,则可能需要直接定义名称的单数和复数形式. sequelize 允许你通过一些参数来执行此操作.

### 定义模型时

模型应以单词的单数形式定义. 例：

```js
sequelize.define('foo', { name: DataTypes.STRING });
```

上面的模型名称是 `foo`(单数),表名称是 `foos`,因为 Sequelize 会自动获取表名称的复数形式.

### 在模型中定义参考键时

```js
sequelize.define('foo', {
  name: DataTypes.STRING,
  barId: {
    type: DataTypes.INTEGER,
    allowNull: false,
    references: {
      model: "bars",
      key: "id"
    },
    onDelete: "CASCADE"
  },
});
```

在上面的示例中,我们手动定义了引用另一个模型的键. 这不是通常的做法,但是如果必须这样做,则应在此使用表名. 这是因为引用是根据引用的表名创建的. 在上面的示例中,使用了复数形式(`bars`),假设 `bar` 模型是使用默认设置创建的(使其基础表自动复数).

### 从预先加载中检索数据时

当你在查询中执行 `include` 时,包含的数据将根据以下规则添加到返回对象的额外字段中：

* 当包含来自单个关联(`hasOne` 或 `belongsTo`)的内容时,字段名称将是模型名称的单数形式;
* 当包含来自多个关联(`hasMany` 或 `belongsToMany`)的内容时,字段名称将是模型的复数形式.

简而言之,在每种情况下,字段名称将采用最合乎逻辑的形式.

示例:

```js
// 假设 Foo.hasMany(Bar)
const foo = Foo.findOne({ include: Bar });
// foo.bars 将是一个数组
// foo.bar 将不存在,因为它没有意义

// 假设 Foo.hasOne(Bar)
const foo = Foo.findOne({ include: Bar });
// foo.bar 将是一个对象(如果没有关联的模型,则可能为 null)
// foo.bars 将不存在,因为它没有意义

// 等等.
```

### 定义别名时覆盖单数和复数

在为关联定义别名时,你可以传递一个对象以指定单数和复数形式,而不仅仅是使用 `{ as: 'myAlias' }`.

```js
Project.belongsToMany(User, {
  as: {
    singular: 'líder',
    plural: 'líderes'
  }
});
```

如果你知道模型在关联中将始终使用相同的别名,则可以将单数和复数形式直接提供给模型本身：

```js
const User = sequelize.define('user', { /* ... */ }, {
  name: {
    singular: 'líder',
    plural: 'líderes',
  }
});
Project.belongsToMany(User);
```

添加到用户实例的混入文件将使用正确的形式. 例如,Sequelize 将代替 `project.addUser()` 来提供 `project.getLíder()`. 另外,Sequelize 将代替 `project.setUsers()` 来提供 `project.setLíderes()`.

注意：记得使用 `as` 来改变关联的名字也会改变外键的名字. 因此,建议也指定在这种情况下直接涉及的外键.

```js
// 错误示例
Invoice.belongsTo(Subscription, { as: 'TheSubscription' });
Subscription.hasMany(Invoice);
```

上面的第一个调用将在 `Invoice` 上建立一个名为 `theSubscriptionId` 的外键. 但是,第二个调用也会在 `Invoice` 上建立外键(因为众所周知, `hasMany` 调用会将外键放置在目标模型中)-但是,它将被命名为 `subscriptionId`. 这样,你将同时具有 `subscriptionId` 和 `theSubscriptionId` 列.

最好的方法是为外键选择一个名称,并将其显式放置在两个调用中. 例如,如果选择了 `subscription_id`：

```js
// 修正示例
Invoice.belongsTo(Subscription, { as: 'TheSubscription', foreignKey: 'subscription_id' });
Subscription.hasMany(Invoice, { foreignKey: 'subscription_id' });
```