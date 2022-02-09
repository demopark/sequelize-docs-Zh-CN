# Associations - 关联

Sequelize 支持标准关联关系: [一对一](https://en.wikipedia.org/wiki/One-to-one_%28data_model%29), [一对多](https://en.wikipedia.org/wiki/One-to-many_%28data_model%29) 和 [多对多](https://en.wikipedia.org/wiki/Many-to-many_%28data_model%29).

为此,Sequelize 提供了 **四种** 关联类型,并将它们组合起来以创建关联：

* `HasOne` 关联类型
* `BelongsTo` 关联类型
* `HasMany` 关联类型
* `BelongsToMany` 关联类型

该指南将讲解如何定义这四种类型的关联,然后讲解如何将它们组合来定义三种标准关联类型([一对一](https://en.wikipedia.org/wiki/One-to-one_%28data_model%29), [一对多](https://en.wikipedia.org/wiki/One-to-many_%28data_model%29) 和 [多对多](https://en.wikipedia.org/wiki/Many-to-many_%28data_model%29)).

## 定义 Sequelize 关联

四种关联类型的定义非常相似. 假设我们有两个模型 `A` 和 `B`. 告诉 Sequelize 两者之间的关联仅需要调用一个函数：

```js
const A = sequelize.define('A', /* ... */);
const B = sequelize.define('B', /* ... */);

A.hasOne(B); // A 有一个 B
A.belongsTo(B); // A 属于 B
A.hasMany(B); // A 有多个 B
A.belongsToMany(B, { through: 'C' }); // A 属于多个 B , 通过联结表 C
```

它们都接受一个对象作为第二个参数(前三个参数是可选的,而对于包含 `through` 属性的 `belongsToMany` 是必需的)：
They all accept an options object as a second parameter

```js
A.hasOne(B, { /* 参数 */ });
A.belongsTo(B, { /* 参数 */ });
A.hasMany(B, { /* 参数 */ });
A.belongsToMany(B, { through: 'C', /* 参数 */ });
```

关联的定义顺序是有关系的. 换句话说,对于这四种情况,定义顺序很重要. 在上述所有示例中,`A` 称为 **源** 模型,而 `B` 称为 **目标** 模型. 此术语很重要.

`A.hasOne(B)` 关联意味着 `A` 和 `B` 之间存在一对一的关系,外键在目标模型(`B`)中定义.

`A.belongsTo(B)`关联意味着 `A` 和 `B` 之间存在一对一的关系,外键在源模型中定义(`A`).

`A.hasMany(B)` 关联意味着 `A` 和 `B` 之间存在一对多关系,外键在目标模型(`B`)中定义.

这三个调用将导致 Sequelize 自动将外键添加到适当的模型中(除非它们已经存在).

`A.belongsToMany(B, { through: 'C' })` 关联意味着将表 `C` 用作[联结表](https://en.wikipedia.org/wiki/Associative_entity),在 `A` 和 `B` 之间存在多对多关系. 具有外键(例如,`aId` 和 `bId`). Sequelize 将自动创建此模型 `C`(除非已经存在),并在其上定义适当的外键.

*注意：在上面的 `belongsToMany` 示例中,字符串(`'C'`)被传递给 `through` 参数. 在这种情况下,Sequelize 会自动使用该名称生成模型. 但是,如果已经定义了模型,也可以直接传递模型.*

这些是每种关联类型中涉及的主要思想. 但是,这些关系通常成对使用,以便 Sequelize 更好地使用. 这将在后文中看到.

## 创建标准关系

如前所述,Sequelize 关联通常成对定义. 综上所述：

* 创建一个 **一对一** 关系, `hasOne` 和 `belongsTo` 关联一起使用;
* 创建一个 **一对多** 关系, `hasMany` he  `belongsTo` 关联一起使用;
* 创建一个 **多对多** 关系, 两个 `belongsToMany` 调用一起使用.
  * 注意: 还有一个 *超级多对多* 关系,一次使用六个关联,将在[高级多对多关系指南](../advanced-association-concepts/advanced-many-to-many.md)中进行讨论.

接下来将进行详细介绍. 本章末尾将讨论使用这些成对而不是单个关联的优点.

## 一对一关系

### 哲理

在深入探讨使用 Sequelize 的各个方面之前,退后一步来考虑一对一关系会发生什么是很有用的.

假设我们有两个模型,`Foo` 和 `Bar`.我们要在 Foo 和 Bar 之间建立一对一的关系.我们知道在关系数据库中,这将通过在其中一个表中建立外键来完成.因此,在这种情况下,一个非常关键的问题是：我们希望该外键在哪个表中？换句话说,我们是要	`Foo` 拥有 `barId` 列,还是 `Bar` 应当拥有 `fooId` 列？

原则上,这两个选择都是在 Foo 和 Bar 之间建立一对一关系的有效方法.但是,当我们说 *"Foo 和 Bar 之间存在一对一关系"* 时,尚不清楚该关系是 *强制性* 的还是可选的.换句话说,Foo 是否可以没有 Bar 而存在？ Foo 的 Bar 可以存在吗？这些问题的答案有助于帮我们弄清楚外键列在哪里.

### 目标

对于本示例的其余部分,我们假设我们有两个模型,即 `Foo` 和 `Bar`. 我们想要在它们之间建立一对一的关系,以便 `Bar` 获得 `fooId` 列.

### 实践

实现该目标的主要设置如下：

```js
Foo.hasOne(Bar);
Bar.belongsTo(Foo);
```

由于未传递任何参数,因此 Sequelize 将从模型名称中推断出要做什么. 在这种情况下,Sequelize 知道必须将 `fooId` 列添加到 `Bar` 中.

这样,在上述代码之后调用 `Bar.sync()` 将产生以下 SQL(例如,在PostgreSQL上)：

```sql
CREATE TABLE IF NOT EXISTS "foos" (
  /* ... */
);
CREATE TABLE IF NOT EXISTS "bars" (
  /* ... */
  "fooId" INTEGER REFERENCES "foos" ("id") ON DELETE SET NULL ON UPDATE CASCADE
  /* ... */
);
```

### 参数

可以将各种参数作为关联调用的第二个参数传递.

#### `onDelete` 和 `onUpdate`

例如,要配置 `ON DELETE` 和 `ON UPDATE`  行为,你可以执行以下操作：

```js
Foo.hasOne(Bar, {
  onDelete: 'RESTRICT',
  onUpdate: 'RESTRICT'
});
Bar.belongsTo(Foo);
```

可用的参数为 `RESTRICT`, `CASCADE`, `NO ACTION`, `SET DEFAULT` 和 `SET NULL`.

一对一关联的默认值, `ON DELETE` 为 `SET NULL` 而 `ON UPDATE` 为 `CASCADE`.

#### 自定义外键

上面显示的 `hasOne` 和 `belongsTo` 调用都会推断出要创建的外键应称为 `fooId`. 如要使用其他名称,例如 `myFooId`：

```js
// 方法 1
Foo.hasOne(Bar, {
  foreignKey: 'myFooId'
});
Bar.belongsTo(Foo);

// 方法 2
Foo.hasOne(Bar, {
  foreignKey: {
    name: 'myFooId'
  }
});
Bar.belongsTo(Foo);

// 方法 3
Foo.hasOne(Bar);
Bar.belongsTo(Foo, {
  foreignKey: 'myFooId'
});

// 方法 4
Foo.hasOne(Bar);
Bar.belongsTo(Foo, {
  foreignKey: {
    name: 'myFooId'
  }
});
```

如上所示,`foreignKey` 参数接受一个字符串或一个对象. 当接收到一个对象时,该对象将用作列的定义,就像在标准的 `sequelize.define` 调用中所做的一样. 因此,指定诸如 `type`, `allowNull`, `defaultValue` 等参数就可以了.

例如,要使用 `UUID` 作为外键数据类型而不是默认值(`INTEGER`),只需执行以下操作：

```js
const { DataTypes } = require("Sequelize");

Foo.hasOne(Bar, {
  foreignKey: {
    // name: 'myFooId'
    type: DataTypes.UUID
  }
});
Bar.belongsTo(Foo);
```

#### 强制性与可选性关联

默认情况下,该关联被视为可选. 换句话说,在我们的示例中,`fooId` 允许为空,这意味着一个 Bar 可以不存在 Foo 而存在. 只需在外键选项中指定 `allowNull: false` 即可更改此设置：

```js
Foo.hasOne(Bar, {
  foreignKey: {
    allowNull: false
  }
});
// "fooId" INTEGER NOT NULL REFERENCES "foos" ("id") ON DELETE RESTRICT ON UPDATE RESTRICT
```

## 一对多关系

### 原理

一对多关联将一个源与多个目标连接,而所有这些目标仅与此单个源连接.

这意味着,与我们必须选择放置外键的一对一关联不同,在一对多关联中只有一个选项. 例如,如果一个 Foo 有很多 Bar(因此每个 Bar 都属于一个 Foo),那么唯一明智的方式就是在 `Bar` 表中有一个 `fooId` 列. 而反过来是不可能的,因为一个 `Foo` 会有很多 `Bar`.

### 目标

在这个例子中,我们有模型 `Team` 和 `Player`. 我们要告诉 Sequelize,他们之间存在一对多的关系,这意味着一个 Team 有 Player ,而每个 Player 都属于一个 Team.

### 实践

这样做的主要方法如下：

```js
Team.hasMany(Player);
Player.belongsTo(Team);
```

同样,实现此目标的主要方法是使用一对 Sequelize 关联(`hasMany` 和 `belongsTo`).

例如,在 PostgreSQL 中,以上设置将在 `sync()` 之后产生以下 SQL：

```sql
CREATE TABLE IF NOT EXISTS "Teams" (
  /* ... */
);
CREATE TABLE IF NOT EXISTS "Players" (
  /* ... */
  "TeamId" INTEGER REFERENCES "Teams" ("id") ON DELETE SET NULL ON UPDATE CASCADE,
  /* ... */
);
```

### 参数

在这种情况下要应用的参数与一对一情况相同. 例如,要更改外键的名称并确保该关系是强制性的,我们可以执行以下操作：

```js
Team.hasMany(Player, {
  foreignKey: 'clubId'
});
Player.belongsTo(Team);
```

如同一对一关系, `ON DELETE` 默认为 `SET NULL` 而 `ON UPDATE` 默认为 `CASCADE`.

## 多对多关系

### 原理

多对多关联将一个源与多个目标相连,而所有这些目标又可以与第一个目标之外的其他源相连.

不能像其他关系那样通过向其中一个表添加一个外键来表示这一点. 取而代之的是使用[联结模型](https://en.wikipedia.org/wiki/Associative_entity)的概念. 这将是一个额外的模型(以及数据库中的额外表),它将具有两个外键列并跟踪关联. 联结表有时也称为 *join table* 或 *through table*.

### 目标

对于此示例,我们将考虑模型 `Movie` 和 `Actor`. 一位 actor 可能参与了许多 movies,而一部 movie 中有许多 actors 参与了其制作. 跟踪关联的联结表将被称为 `ActorMovies`,其中将包含外键 `movieId` 和 `actorId`.

### 实践

在 Sequelize 中执行此操作的主要方法如下：

```js
const Movie = sequelize.define('Movie', { name: DataTypes.STRING });
const Actor = sequelize.define('Actor', { name: DataTypes.STRING });
Movie.belongsToMany(Actor, { through: 'ActorMovies' });
Actor.belongsToMany(Movie, { through: 'ActorMovies' });
```

因为在 `belongsToMany` 的 `through` 参数中给出了一个字符串,所以 Sequelize 将自动创建 `ActorMovies` 模型作为联结模型. 例如,在 PostgreSQL 中：

```sql
CREATE TABLE IF NOT EXISTS "ActorMovies" (
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "MovieId" INTEGER REFERENCES "Movies" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
  "ActorId" INTEGER REFERENCES "Actors" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
  PRIMARY KEY ("MovieId","ActorId")
);
```

除了字符串以外,还支持直接传递模型,在这种情况下,给定的模型将用作联结模型(并且不会自动创建任何模型). 例如：

```js
const Movie = sequelize.define('Movie', { name: DataTypes.STRING });
const Actor = sequelize.define('Actor', { name: DataTypes.STRING });
const ActorMovies = sequelize.define('ActorMovies', {
  MovieId: {
    type: DataTypes.INTEGER,
    references: {
      model: Movie, // 'Movies' 也可以使用
      key: 'id'
    }
  },
  ActorId: {
    type: DataTypes.INTEGER,
    references: {
      model: Actor, // 'Actors' 也可以使用
      key: 'id'
    }
  }
});
Movie.belongsToMany(Actor, { through: ActorMovies });
Actor.belongsToMany(Movie, { through: ActorMovies });
```

上面的代码在 PostgreSQL 中产生了以下 SQL,与上面所示的代码等效：

```sql
CREATE TABLE IF NOT EXISTS "ActorMovies" (
  "MovieId" INTEGER NOT NULL REFERENCES "Movies" ("id") ON DELETE RESTRICT ON UPDATE CASCADE,
  "ActorId" INTEGER NOT NULL REFERENCES "Actors" ("id") ON DELETE RESTRICT ON UPDATE CASCADE,
  "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
  UNIQUE ("MovieId", "ActorId"),     -- 注意: Sequelize 产生了这个 UNIQUE 约束,但是
  PRIMARY KEY ("MovieId","ActorId")  -- 这没有关系,因为它也是 PRIMARY KEY
);
```

### 参数

与一对一和一对多关系不同,对于多对多关系,`ON UPDATE` 和 `ON DELETE` 的默认值为 `CASCADE`.

当模型中不存在主键时，Belongs-to-Many 将创建一个唯一键. 可以使用 **uniqueKey**  参数覆盖此唯一键名. 若不希望产生唯一键, 可以使用 ***unique: false*** 参数.

```js
Project.belongsToMany(User, { through: UserProjects, uniqueKey: 'my_custom_unique' })
```

## 基本的涉及关联的查询

了解了定义关联的基础知识之后,我们可以查看涉及关联的查询. 最常见查询是 *read* 查询(即 SELECT). 稍后,将展示其他类型的查询.

为了研究这一点,我们将思考一个例子,其中有船和船长,以及它们之间的一对一关系. 我们将在外键上允许 null(默认值),这意味着船可以在没有船长的情况下存在,反之亦然.

```js
// 这是我们用于以下示例的模型的设置
const Ship = sequelize.define('ship', {
  name: DataTypes.STRING,
  crewCapacity: DataTypes.INTEGER,
  amountOfSails: DataTypes.INTEGER
}, { timestamps: false });
const Captain = sequelize.define('captain', {
  name: DataTypes.STRING,
  skillLevel: {
    type: DataTypes.INTEGER,
    validate: { min: 1, max: 10 }
  }
}, { timestamps: false });
Captain.hasOne(Ship);
Ship.belongsTo(Captain);
```

### 获取关联 - 预先加载 vs 延迟加载

预先加载和延迟加载的概念是理解获取关联如何在 Sequelize 中工作的基础. 延迟加载是指仅在确实需要时才获取关联数据的技术. 另一方面,预先加载是指从一开始就通过较大的查询一次获取所有内容的技术.

#### 延迟加载示例

```js
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  }
});
// 用获取到的 captain 做点什么
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
// 现在我们需要有关他的 ship 的信息!
const hisShip = await awesomeCaptain.getShip();
// 用 ship 做点什么
console.log('Ship Name:', hisShip.name);
console.log('Amount of Sails:', hisShip.amountOfSails);
```

请注意,在上面的示例中,我们进行了两个查询,仅在要使用它时才获取关联的 ship. 如果我们可能需要也可能不需要这艘 ship,或者我们只想在少数情况下有条件地取回它,这会特别有用; 这样,我们可以仅在必要时提取,从而节省时间和内存.

注意：上面使用的 `getShip()` 实例方法是 Sequelize 自动添加到 Captain 实例的方法之一. 还有其他方法, 你将在本指南的后面部分进一步了解它们.

#### 预先加载示例

```js
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  },
  include: Ship
});
// 现在 ship 跟着一起来了
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
console.log('Ship Name:', awesomeCaptain.ship.name);
console.log('Amount of Sails:', awesomeCaptain.ship.amountOfSails);
```

如上所示,通过使用 include 参数 在 Sequelize 中执行预先加载. 观察到这里只对数据库执行了一个查询(与实例一起带回关联的数据).

这只是 Sequelize 中预先加载的简单介绍. 还有更多内容,你可以在[预先加载的专用指南](../advanced-association-concepts/eager-loading.md)中学习

### 创建, 更新和删除

上面显示了查询有关关联的数据的基础知识. 对于创建,更新和删除,你可以：

* 直接使用标准模型查询：

  ```js
  // 示例：使用标准方法创建关联的模型
  Bar.create({
    name: 'My Bar',
    fooId: 5
  });
  // 这将创建一个属于 ID 5 的 Foo 的 Bar
  // 这里没有什么特别的东西
  ```

* 或使用关联模型可用的 *[特殊方法/混合](＃special-methods-mixins-to-instances)* ,这将在本文稍后进行解释.

**注意:** [`save()`实例方法](https://sequelize.org/master/class/lib/model.js~Model.html#instance-method-save) 并不知道关联关系. 如果你修改了 *父级* 对象预先加载的 *子级* 的值,那么在父级上调用 `save()` 将会忽略子级上发生的修改.

## 关联别名 & 自定义外键

在以上所有示例中,Sequelize 自动定义了外键名称. 例如,在船和船长示例中,Sequelize 在 Ship 模型上自动定义了一个 `captainId` 字段. 然而,想要自定义外键也是很容易的.

让我们以简化的形式考虑 Ship 和 Captain 模型,仅着眼于当前主题,如下所示(较少的字段)：

```js
const Ship = sequelize.define('ship', { name: DataTypes.STRING }, { timestamps: false });
const Captain = sequelize.define('captain', { name: DataTypes.STRING }, { timestamps: false });
```

有三种方法可以为外键指定不同的名称：

* 通过直接提供外键名称
* 通过定义别名
* 通过两个方法同时进行

### 回顾: 默认设置

通过简单地使用 `Ship.belongsTo(Captain)`,sequelize 将自动生成外键名称：

```js
Ship.belongsTo(Captain); // 这将在 Ship 中创建 `captainId` 外键.

// 通过将模型传递给 `include` 来完成预先加载:
console.log((await Ship.findAll({ include: Captain })).toJSON());
// 或通过提供关联的模型名称:
console.log((await Ship.findAll({ include: 'captain' })).toJSON());

// 同样,实例获得用于延迟加载的 `getCaptain()` 方法：
const ship = Ship.findOne();
console.log((await ship.getCaptain()).toJSON());
```

### 直接提供外键名称

可以直接在关联定义的参数中提供外键名称,如下所示：

```js
Ship.belongsTo(Captain, { foreignKey: 'bossId' }); // 这将在 Ship 中创建 `bossId` 外键.

// 通过将模型传递给 `include` 来完成预先加载:
console.log((await Ship.findAll({ include: Captain })).toJSON());
// 或通过提供关联的模型名称:
console.log((await Ship.findAll({ include: 'Captain' })).toJSON());

// 同样,实例获得用于延迟加载的 `getCaptain()` 方法:
const ship = Ship.findOne();
console.log((await ship.getCaptain()).toJSON());
```

### 定义别名

定义别名比简单指定外键的自定义名称更强大. 通过一个示例可以更好地理解这一点：

<!-- 注意：这部分的任何更改也可能需要对 Advanced-many-to-many.md 进行更改 -->

```js
Ship.belongsTo(Captain, { as: 'leader' }); // 这将在 Ship 中创建 `leaderId` 外键.

// 通过将模型传递给 `include` 不能再触发预先加载:
console.log((await Ship.findAll({ include: Captain })).toJSON()); // 引发错误
// 相反,你必须传递别名:
console.log((await Ship.findAll({ include: 'leader' })).toJSON());
// 或者,你可以传递一个指定模型和别名的对象:
console.log((await Ship.findAll({
  include: {
    model: Captain,
    as: 'leader'
  }
})).toJSON());

// 同样,实例获得用于延迟加载的 `getLeader()`方法:
const ship = Ship.findOne();
console.log((await ship.getLeader()).toJSON());
```

当你需要在同一模型之间定义两个不同的关联时,别名特别有用. 例如,如果我们有`Mail` 和 `Person` 模型,则可能需要将它们关联两次,以表示邮件的 `sender` 和 `receiver`. 在这种情况下,我们必须为每个关联使用别名,因为否则,诸如 `mail.getPerson()` 之类的调用将是模棱两可的. 使用 `sender` 和 `receiver` 别名,我们将有两种可用的可用方法：`mail.getSender()` 和 `mail.getReceiver()`,它们都返回一个`Promise<Person>`.

在为 `hasOne` 或 `belongsTo` 关联定义别名时,应使用单词的单数形式(例如上例中的 `leader`). 另一方面,在为 `hasMany` 和 `belongsToMany` 定义别名时,应使用复数形式. [高级多对多关联指南](advanced-association-concepts/advanced-many-to-many.md)中介绍了定义多对多关系(带有`belongsToMany`)的别名.

### 两者都做

我们可以定义别名,也可以直接定义外键:

```js
Ship.belongsTo(Captain, { as: 'leader', foreignKey: 'bossId' }); // 这将在 Ship 中创建 `bossId` 外键.

// 由于定义了别名,因此仅通过将模型传递给 `include`,预先加载将不起作用:
console.log((await Ship.findAll({ include: Captain })).toJSON()); // 引发错误
// 相反,你必须传递别名:
console.log((await Ship.findAll({ include: 'leader' })).toJSON());
// 或者,你可以传递一个指定模型和别名的对象:
console.log((await Ship.findAll({
  include: {
    model: Captain,
    as: 'leader'
  }
})).toJSON());

// 同样,实例获得用于延迟加载的 `getLeader()` 方法:
const ship = Ship.findOne();
console.log((await ship.getLeader()).toJSON());
```

## 添加到实例的特殊方法

当两个模型之间定义了关联时,这些模型的实例将获得特殊的方法来与其关联的另一方进行交互.

例如,如果我们有两个模型 `Foo` 和 `Bar`,并且它们是关联的,则它们的实例将具有以下可用的方法,具体取决于关联类型：

### `Foo.hasOne(Bar)`

* `fooInstance.getBar()`
* `fooInstance.setBar()`
* `fooInstance.createBar()`

示例:

```js
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBar()); // null
await foo.setBar(bar1);
console.log((await foo.getBar()).name); // 'some-bar'
await foo.createBar({ name: 'yet-another-bar' });
const newlyAssociatedBar = await foo.getBar();
console.log(newlyAssociatedBar.name); // 'yet-another-bar'
await foo.setBar(null); // Un-associate
console.log(await foo.getBar()); // null
```

### `Foo.belongsTo(Bar)`

来自 `Foo.hasOne(Bar)` 的相同内容:

* `fooInstance.getBar()`
* `fooInstance.setBar()`
* `fooInstance.createBar()`

### `Foo.hasMany(Bar)`

* `fooInstance.getBars()`
* `fooInstance.countBars()`
* `fooInstance.hasBar()`
* `fooInstance.hasBars()`
* `fooInstance.setBars()`
* `fooInstance.addBar()`
* `fooInstance.addBars()`
* `fooInstance.removeBar()`
* `fooInstance.removeBars()`
* `fooInstance.createBar()`

示例:

```js
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBars()); // []
console.log(await foo.countBars()); // 0
console.log(await foo.hasBar(bar1)); // false
await foo.addBars([bar1, bar2]);
console.log(await foo.countBars()); // 2
await foo.addBar(bar1);
console.log(await foo.countBars()); // 2
console.log(await foo.hasBar(bar1)); // true
await foo.removeBar(bar2);
console.log(await foo.countBars()); // 1
await foo.createBar({ name: 'yet-another-bar' });
console.log(await foo.countBars()); // 2
await foo.setBars([]); // 取消关联所有先前关联的 Bars
console.log(await foo.countBars()); // 0
```

getter 方法接受参数,就像通常的 finder 方法(例如`findAll`)一样：

```js
const easyTasks = await project.getTasks({
  where: {
    difficulty: {
      [Op.lte]: 5
    }
  }
});
const taskTitles = (await project.getTasks({
  attributes: ['title'],
  raw: true
})).map(task => task.title);
```

### `Foo.belongsToMany(Bar, { through: Baz })`

来自 `Foo.hasMany(Bar)` 的相同内容:

* `fooInstance.getBars()`
* `fooInstance.countBars()`
* `fooInstance.hasBar()`
* `fooInstance.hasBars()`
* `fooInstance.setBars()`
* `fooInstance.addBar()`
* `fooInstance.addBars()`
* `fooInstance.removeBar()`
* `fooInstance.removeBars()`
* `fooInstance.createBar()`

对于 belongsToMany 关系, 默认情况下, `getBars()` 将返回连接表中的所有字段. 请注意, 任何 `include` 参数都将应用于目标 `Bar` 对象, 因此无法像使用 `find` 方法进行预加载时那样尝试为连接表设置参数. 要选择要包含的连接表的哪些属性, `getBars()` 支持一个 `joinTableAttributes` 选项, 其使用类似于在 `include` 中设置 `through.attributes`.  例如, 设定 Foo belongsToMany Bar, 以下都将输出没有连接表字段的结果:

```js
const foo = Foo.findByPk(id, {
  include: [{
    model: Bar,
    through: { attributes: [] }
  }]
})
console.log(foo.bars)

const foo = Foo.findByPk(id)
console.log(foo.getBars({ joinTableAttributes: [] }))
```


### 注意: 方法名称

如上面的示例所示,Sequelize 赋予这些特殊方法的名称是由前缀(例如,get,add,set)和模型名称(首字母大写)组成的. 必要时,可以使用复数形式,例如在 `fooInstance.setBars()` 中. 同样,不规则复数也由 Sequelize 自动处理. 例如,`Person` 变成 `People` 或者 `Hypothesis` 变成 `Hypotheses`.

如果定义了别名,则将使用别名代替模型名称来形成方法名称. 例如：

```js
Task.hasOne(User, { as: 'Author' });
```

* `taskInstance.getAuthor()`
* `taskInstance.setAuthor()`
* `taskInstance.createAuthor()`

## 为什么关联是成对定义的？

如前所述,就像上面大多数示例中展示的,Sequelize 中的关联通常成对定义：

* 创建一个 **一对一** 关系, `hasOne` 和 `belongsTo` 关联一起使用;
* 创建一个 **一对多** 关系, `hasMany` he  `belongsTo` 关联一起使用;
* 创建一个 **多对多** 关系, 两个 `belongsToMany` 调用一起使用.

当在两个模型之间定义了 Sequelize 关联时,只有 *源* 模型 *知晓关系*. 因此,例如,当使用 `Foo.hasOne(Bar)`(当前,`Foo` 是源模型,而 `Bar` 是目标模型)时,只有 `Foo` 知道该关联的存在. 这就是为什么在这种情况下,如上所示,`Foo` 实例获得方法 `getBar()`, `setBar()` 和 `createBar()` 而另一方面,`Bar` 实例却没有获得任何方法.

类似地,对于 `Foo.hasOne(Bar)`,由于 `Foo` 了解这种关系,我们可以像 `Foo.findOne({ include: Bar })` 中那样执行预先加载,但不能执行 `Bar.findOne({ include: Foo })`.

因此,为了充分发挥 Sequelize 的作用,我们通常成对设置关系,以便两个模型都 *互相知晓*.

实际示范:

* 如果我们未定义关联对,则仅调用 `Foo.hasOne(Bar)`:

  ```js
  // 这有效...
  await Foo.findOne({ include: Bar });

  // 但这会引发错误:
  await Bar.findOne({ include: Foo });
  // SequelizeEagerLoadingError: foo is not associated to bar!
  ```

* 如果我们按照建议定义关联对, 即, `Foo.hasOne(Bar)` 和 `Bar.belongsTo(Foo)`:

  ```js
  // 这有效
  await Foo.findOne({ include: Bar });

  // 这也有效!
  await Bar.findOne({ include: Foo });
  ```

## 涉及相同模型的多个关联

在 Sequelize 中,可以在同一模型之间定义多个关联. 你只需要为它们定义不同的别名：

```js
Team.hasOne(Game, { as: 'HomeTeam', foreignKey: 'homeTeamId' });
Team.hasOne(Game, { as: 'AwayTeam', foreignKey: 'awayTeamId' });
Game.belongsTo(Team);
```

## 创建引用非主键字段的关联

在以上所有示例中,通过引用所涉及模型的主键(在我们的示例中为它们的ID)定义了关联. 但是,Sequelize 允许你定义一个关联,该关联使用另一个字段而不是主键字段来建立关联.

此其他字段必须对此具有唯一的约束(否则,这将没有意义).

### 对于 `belongsTo` 关系

首先,回想一下 `A.belongsTo(B)` 关联将外键放在 *源模型* 中(即,在 `A` 中).

让我们再次使用"船和船长"的示例. 此外,我们将假定船长姓名是唯一的：

```js
const Ship = sequelize.define('ship', { name: DataTypes.STRING }, { timestamps: false });
const Captain = sequelize.define('captain', {
  name: { type: DataTypes.STRING, unique: true }
}, { timestamps: false });
```

这样,我们不用在 Ship 上保留 `captainId`,而是可以保留 `captainName` 并将其用作关联跟踪器. 换句话说,我们的关系将引用目标模型上的另一列：`name` 列,而不是从目标模型(Captain)中引用 `id`. 为了说明这一点,我们必须定义一个 *目标键*. 我们还必须为外键本身指定一个名称：

```js
Ship.belongsTo(Captain, { targetKey: 'name', foreignKey: 'captainName' });
// 这将在源模型(Ship)中创建一个名为 `captainName` 的外键,
// 该外键引用目标模型(Captain)中的 `name` 字段.
```

现在我们可以做类似的事情:

```js
await Captain.create({ name: "Jack Sparrow" });
const ship = await Ship.create({ name: "Black Pearl", captainName: "Jack Sparrow" });
console.log((await ship.getCaptain()).name); // "Jack Sparrow"
```

### 对于 `hasOne` 和 `hasMany` 关系

可以将完全相同的想法应用于 `hasOne` 和 `hasMany`  关联,但是在定义关联时,我们提供了 `sourceKey`,而不是提供 `targetKey`. 这是因为与 `belongsTo` 不同,`hasOne` 和 `hasMany`关联将外键保留在目标模型上：

```js
const Foo = sequelize.define('foo', {
  name: { type: DataTypes.STRING, unique: true }
}, { timestamps: false });
const Bar = sequelize.define('bar', {
  title: { type: DataTypes.STRING, unique: true }
}, { timestamps: false });
const Baz = sequelize.define('baz', { summary: DataTypes.STRING }, { timestamps: false });
Foo.hasOne(Bar, { sourceKey: 'name', foreignKey: 'fooName' });
Bar.hasMany(Baz, { sourceKey: 'title', foreignKey: 'barTitle' });
// [...]
await Bar.setFoo("Foo's Name Here");
await Baz.addBar("Bar's Title Here");
```

### 对于 `belongsToMany` 关系

同样的想法也可以应用于 `belongsToMany` 关系. 但是,与其他情况下(其中只涉及一个外键)不同,`belongsToMany` 关系涉及两个外键,这些外键保留在额外的表(联结表)上.

请考虑以下设置：

```js
const Foo = sequelize.define('foo', {
  name: { type: DataTypes.STRING, unique: true }
}, { timestamps: false });
const Bar = sequelize.define('bar', {
  title: { type: DataTypes.STRING, unique: true }
}, { timestamps: false });
```

有四种情况需要考虑：

* 我们可能希望使用默认的主键为 `Foo` 和 `Bar` 进行多对多关系：

```js
Foo.belongsToMany(Bar, { through: 'foo_bar' });
// 这将创建具有字段 `fooId` 和 `barID` 的联结表 `foo_bar`.
```

* 我们可能希望使用默认主键 `Foo` 的多对多关系,但使用 `Bar` 的不同字段：

```js
Foo.belongsToMany(Bar, { through: 'foo_bar', targetKey: 'title' });
// 这将创建具有字段 `fooId` 和 `barTitle` 的联结表 `foo_bar`.
```

* 我们可能希望使用 `Foo` 的不同字段和 `Bar` 的默认主键进行多对多关系：

```js
Foo.belongsToMany(Bar, { through: 'foo_bar', sourceKey: 'name' });
// 这将创建具有字段 `fooName` 和 `barId` 的联结表 `foo_bar`.
```

* 我们可能希望使用不同的字段为 `Foo` 和 `Bar` 使用多对多关系：

```js
Foo.belongsToMany(Bar, { through: 'foo_bar', sourceKey: 'name', targetKey: 'title' });
// 这将创建带有字段 `fooName` 和 `barTitle` 的联结表 `foo_bar`.
```

### 注意

不要忘记关联中引用的字段必须具有唯一性约束. 否则,将引发错误(对于 SQLite 有时还会发出诡异的错误消息,例如 `SequelizeDatabaseError: SQLITE_ERROR: foreign key mismatch - "ships" referencing "captains"`).

在 `sourceKey` 和 `targetKey` 之间做出决定的技巧只是记住每个关系在何处放置其外键. 如本指南开头所述：

* `A.belongsTo(B)` 将外键保留在源模型中(`A`),因此引用的键在目标模型中,因此使用了 `targetKey`.

* `A.hasOne(B)` 和 `A.hasMany(B)` 将外键保留在目标模型(`B`)中,因此引用的键在源模型中,因此使用了 `sourceKey`.

* `A.belongsToMany(B)` 包含一个额外的表(联结表),因此 `sourceKey` 和 `targetKey` 均可用,其中 `sourceKey` 对应于`A`(源)中的某个字段而 `targetKey` 对应于 `B`(目标)中的某个字段.
