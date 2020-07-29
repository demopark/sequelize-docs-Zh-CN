# Advanced M:N Associations - 高级 M:N 关联

阅读本指南之前,请确保已阅读 [关联指南](core-concepts/assocs.md).

让我们从  `User` 和 `Profile` 之间的多对多关系示例开始.

```js
const User = sequelize.define('user', {
  username: DataTypes.STRING,
  points: DataTypes.INTEGER
}, { timestamps: false });
const Profile = sequelize.define('profile', {
  name: DataTypes.STRING
}, { timestamps: false });
```

定义多对多关系的最简单方法是：

```js
User.belongsToMany(Profile, { through: 'User_Profiles' });
Profile.belongsToMany(User, { through: 'User_Profiles' });
```

通过将字符串传递给上面的 `through`,我们要求 Sequelize 自动生成名为 `User_Profiles` 的模型作为 *联结表*,该模型只有两列： `userId` 和 `profileId`. 在这两个列上将建立一个复合唯一键.

我们还可以为自己定义一个模型,以用作联结表.

```js
const User_Profile = sequelize.define('User_Profile', {}, { timestamps: false });
User.belongsToMany(Profile, { through: User_Profile });
Profile.belongsToMany(User, { through: User_Profile });
```

以上具有完全相同的效果. 注意,我们没有在 `User_Profile` 模型上定义任何属性. 我们将其传递给 `belongsToMany` 调用的事实告诉 sequelize 自动创建两个属性 `userId` 和 `profileId`,就像其他关联一样,也会导致 Sequelize 自动向其中一个涉及的模型添加列.

然而,自己定义模型有几个优点. 例如,我们可以在联结表中定义更多列：

```js
const User_Profile = sequelize.define('User_Profile', {
  selfGranted: DataTypes.BOOLEAN
}, { timestamps: false });
User.belongsToMany(Profile, { through: User_Profile });
Profile.belongsToMany(User, { through: User_Profile });
```

这样,我们现在可以在联结表中跟踪额外的信息,即 `selfGranted` 布尔值. 例如,当调用 `user.addProfile()` 时,我们可以使用 `through` 参数传递额外列的值.

示例:

```js
const amidala = User.create({ username: 'p4dm3', points: 1000 });
const queen = Profile.create({ name: 'Queen' });
await amidala.addProfile(queen, { through: { selfGranted: false } });
const result = await User.findOne({
  where: { username: 'p4dm3' },
  include: Profile
});
console.log(result);
```

输出:

```json
{
  "id": 4,
  "username": "p4dm3",
  "points": 1000,
  "profiles": [
    {
      "id": 6,
      "name": "queen",
      "User_Profile": {
        "userId": 4,
        "profileId": 6,
        "selfGranted": false
      }
    }
  ]
}
```

你也可以在单个 `create` 调用中创建所有关系.

示例:

```js
const amidala = await User.create({
  username: 'p4dm3',
  points: 1000,
  profiles: [{
    name: 'Queen',
    User_Profile: {
      selfGranted: true
    }
  }]
}, {
  include: Profile
});

const result = await User.findOne({
  where: { username: 'p4dm3' },
  include: Profile
});

console.log(result);
```

输出:

```json
{
  "id": 1,
  "username": "p4dm3",
  "points": 1000,
  "profiles": [
    {
      "id": 1,
      "name": "Queen",
      "User_Profile": {
        "selfGranted": true,
        "userId": 1,
        "profileId": 1
      }
    }
  ]
}
```

你可能已经注意到 `User_Profiles` 表中没有 `id` 字段. 如上所述,它具有复合唯一键. 该复合唯一密钥的名称由 Sequelize 自动选择,但可以使用 `uniqueKey` 参数进行自定义：

```js
User.belongsToMany(Profile, { through: User_Profiles, uniqueKey: 'my_custom_unique' });
```

如果需要的话,另一种可能是强制联结表像其他标准表一样具有主键. 为此,只需在模型中定义主键:

```js
const User_Profile = sequelize.define('User_Profile', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  },
  selfGranted: DataTypes.BOOLEAN
}, { timestamps: false });
User.belongsToMany(Profile, { through: User_Profile });
Profile.belongsToMany(User, { through: User_Profile });
```

上面的代码当然仍然会创建两列 `userId` 和 `profileId`,但是模型不会在其上设置复合唯一键,而是将其 `id` 列用作主键. 其他一切仍然可以正常工作.

## 联结表与普通表以及"超级多对多关联"

现在,我们将比较上面显示的最后一个"多对多"设置与通常的"一对多"关系的用法,以便最后得出 *超级多对多关系* 的概念作为结论.

### 模型回顾 (有少量重命名)

为了使事情更容易理解,让我们将 `User_Profile` 模型重命名为 `grant`. 请注意,所有操作均与以前相同. 我们的模型是：

```js
const User = sequelize.define('user', {
  username: DataTypes.STRING,
  points: DataTypes.INTEGER
}, { timestamps: false });

const Profile = sequelize.define('profile', {
  name: DataTypes.STRING
}, { timestamps: false });

const Grant = sequelize.define('grant', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  },
  selfGranted: DataTypes.BOOLEAN
}, { timestamps: false });
```

我们使用 `Grant` 模型作为联结表在 `User` 和 `Profile` 之间建立了多对多关系：

```js
User.belongsToMany(Profile, { through: Grant });
Profile.belongsToMany(User, { through: Grant });
```

这会自动将 `userId` 和 `profileId` 列添加到 `Grant` 模型中.

**注意:** 如上所示,我们选择强制 `grant` 模型具有单个主键(通常称为 `id`). 对于 *超级多对多关系*(即将定义),这是必需的.

### 改用一对多关系

除了建立上面定义的多对多关系之外,如果我们执行以下操作怎么办？

```js
// 在 User 和 Grant 之间设置一对多关系
User.hasMany(Grant);
Grant.belongsTo(User);

// 在Profile 和 Grant 之间也设置一对多关系
Profile.hasMany(Grant);
Grant.belongsTo(Profile);
```

结果基本相同！ 这是因为 `User.hasMany(Grant)` 和 `Profile.hasMany(Grant)` 会分别自动将 `userId` 和 `profileId` 列添加到  `Grant` 中.

这表明一个多对多关系与两个一对多关系没有太大区别. 数据库中的表看起来相同.

唯一的区别是你尝试使用 Sequelize 执行预先加载时.

```js
// 使用多对多方法,你可以:
User.findAll({ include: Profile });
Profile.findAll({ include: User });
// However, you can't do:
User.findAll({ include: Grant });
Profile.findAll({ include: Grant });
Grant.findAll({ include: User });
Grant.findAll({ include: Profile });

// 另一方面,通过双重一对多方法,你可以:
User.findAll({ include: Grant });
Profile.findAll({ include: Grant });
Grant.findAll({ include: User });
Grant.findAll({ include: Profile });
// However, you can't do:
User.findAll({ include: Profile });
Profile.findAll({ include: User });
// 尽管你可以使用嵌套 include 来模拟那些,如下所示:
User.findAll({
  include: {
    model: Grant,
    include: Profile
  }
}); // 这模拟了 `User.findAll({ include: Profile })`,
    // 但是生成的对象结构有些不同.
    // 原始结构的格式为 `user.profiles[].grant`,
    // 而模拟结构的格式为 `user.grants[].profiles[]`.
```

### 两全其美：超级多对多关系

我们可以简单地组合上面显示的两种方法！

```js
// 超级多对多关系
User.belongsToMany(Profile, { through: Grant });
Profile.belongsToMany(User, { through: Grant });
User.hasMany(Grant);
Grant.belongsTo(User);
Profile.hasMany(Grant);
Grant.belongsTo(Profile);
```

这样,我们可以进行各种预先加载：

```js
// 全部可以使用:
User.findAll({ include: Profile });
Profile.findAll({ include: User });
User.findAll({ include: Grant });
Profile.findAll({ include: Grant });
Grant.findAll({ include: User });
Grant.findAll({ include: Profile });
```

我们甚至可以执行各种深层嵌套的 include：

```js
User.findAll({
  include: [
    {
      model: Grant,
      include: [User, Profile]
    },
    {
      model: Profile,
      include: {
        model: User,
        include: {
          model: Grant,
          include: [User, Profile]
        }
      }
    }
  ]
});
```

## 别名和自定义键名

与其他关系类似,可以为多对多关系定义别名.

在继续之前,请回顾[关联指南](core-concepts/assocs.md)上的 `belongsTo` 别名示例. 请注意,在这种情况下,定义关联影响 include 完成方式(即传递关联名称)和 Sequelize 为外键选择的名称(在该示例中,`leaderId` 是在 `Ship` 模型上创建的) .

为一个 `belongsToMany` 关联定义一个别名也会影响 include 执行的方式：

```js
Product.belongsToMany(Category, { as: 'groups', through: 'product_categories' });
Category.belongsToMany(Product, { as: 'items', through: 'product_categories' });

// [...]

await Product.findAll({ include: Category }); // 这无法使用

await Product.findAll({ // 通过别名这可以使用
  include: {
    model: Category,
    as: 'groups'
  }
});

await Product.findAll({ include: 'groups' }); // 这也可以使用
```

但是,在此处定义别名与外键名称无关. 联结表中创建的两个外键的名称仍由 Sequelize 基于关联的模型的名称构造. 通过检查上面示例中的穿透表生成的 SQL,可以很容易看出这一点：

```sql
CREATE TABLE IF NOT EXISTS `product_categories` (
  `createdAt` DATETIME NOT NULL,
  `updatedAt` DATETIME NOT NULL,
  `productId` INTEGER NOT NULL REFERENCES `products` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  `categoryId` INTEGER NOT NULL REFERENCES `categories` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  PRIMARY KEY (`productId`, `categoryId`)
);
```

我们可以看到外键是 `productId` 和 `categoryId`. 要更改这些名称,Sequelize 分别接受参数 `foreignKey` 和 `otherKey`(即,`foreignKey` 定义联结关系中源模型的 key,而 `otherKey` 定义目标模型中的 key)：

```js
Product.belongsToMany(Category, {
  through: 'product_categories',
  foreignKey: 'objectId', // 替换 `productId`
  otherKey: 'typeId' // 替换 `categoryId`
});
Category.belongsToMany(Product, {
  through: 'product_categories',
  foreignKey: 'typeId', // 替换 `categoryId`
  otherKey: 'objectId' // 替换 `productId`
});
```

生成 SQL:

```sql
CREATE TABLE IF NOT EXISTS `product_categories` (
  `createdAt` DATETIME NOT NULL,
  `updatedAt` DATETIME NOT NULL,
  `objectId` INTEGER NOT NULL REFERENCES `products` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  `typeId` INTEGER NOT NULL REFERENCES `categories` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  PRIMARY KEY (`objectId`, `typeId`)
);
```

如上所示,当使用两个 `belongsToMany` 调用定义多对多关系时(这是标准方式),应在两个调用中适当地提供 `foreignKey` 和 `otherKey` 参数. 如果仅在一个调用中传递这些参数,那么 Sequelize 行为将不可靠.

## 自参照

Sequelize 直观地支持自参照多对多关系：

```js
Person.belongsToMany(Person, { as: 'Children', through: 'PersonChildren' })
// 这将创建表 PersonChildren,该表存储对象的 ID.
```

## 从联结表中指定属性

默认情况下,当预先加载多对多关系时,Sequelize 将以以下结构返回数据(基于本指南中的第一个示例)：

```json
// User.findOne({ include: Profile })
{
  "id": 4,
  "username": "p4dm3",
  "points": 1000,
  "profiles": [
    {
      "id": 6,
      "name": "queen",
      "grant": {
        "userId": 4,
        "profileId": 6,
        "selfGranted": false
      }
    }
  ]
}
```

注意,外部对象是一个 `User`,它具有一个名为 `profiles` 的字段,该字段是 `Profile` 数组,因此每个 `Profile` 都带有一个名为 `grant` 的额外字段,这是一个 `Grant` 实例.当从多对多关系预先加载时,这是 Sequelize 创建的默认结构.

但是,如果只需要联结表的某些属性,则可以在 `attributes` 参数中为数组提供所需的属性. 例如,如果只需要穿透表中的 `selfGranted` 属性：

```js
User.findOne({
  include: {
    model: Profile,
    through: {
      attributes: ['selfGranted']
    }
  }
});
```

输出:

```json
{
  "id": 4,
  "username": "p4dm3",
  "points": 1000,
  "profiles": [
    {
      "id": 6,
      "name": "queen",
      "grant": {
        "selfGranted": false
      }
    }
  ]
}
```

如果你根本不想使用嵌套的 `grant` 字段,请使用 `attributes: []`：

```js
User.findOne({
  include: {
    model: Profile,
    through: {
      attributes: []
    }
  }
});
```

输出:

```json
{
  "id": 4,
  "username": "p4dm3",
  "points": 1000,
  "profiles": [
    {
      "id": 6,
      "name": "queen"
    }
  ]
}
```

如果你使用 mixins(例如 `user.getProfiles()`)而不是查找器方法(例如 `User.findAll()`),则必须使用 `joinTableAttributes` 参数：

```js
someUser.getProfiles({ joinTableAttributes: ['selfGranted'] });
```

输出:

```json
[
  {
    "id": 6,
    "name": "queen",
    "grant": {
      "selfGranted": false
    }
  }
]
```

## 多对多对多关系及更多

思考你正在尝试为游戏锦标赛建模. 有玩家和团队. 团队玩游戏. 然而,玩家可以在锦标赛中(但不能在比赛中间)更换团队. 因此,给定一个特定的游戏,有某些团队参与该游戏,并且每个团队都有一组玩家(针对该游戏).

因此,我们首先定义三个相关模型：

```js
const Player = sequelize.define('Player', { username: DataTypes.STRING });
const Team = sequelize.define('Team', { name: DataTypes.STRING });
const Game = sequelize.define('Game', { name: DataTypes.INTEGER });
```

现在的问题是：如何关联它们？

首先,我们注意到：

* 一个游戏有许多与之相关的团队(正在玩该游戏的团队);
* 一个团队可能参加了许多比赛.

以上观察表明,我们需要在 Game 和 Team 之间建立多对多关系. 让我们使用本指南前面解释的超级多对多关系：

```js
// Game 与 Team 之间的超级多对多关系
const GameTeam = sequelize.define('GameTeam', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  }
});
Team.belongsToMany(Game, { through: GameTeam });
Game.belongsToMany(Team, { through: GameTeam });
GameTeam.belongsTo(Game);
GameTeam.belongsTo(Team);
Game.hasMany(GameTeam);
Team.hasMany(GameTeam);
```

关于玩家的部分比较棘手. 我们注意到,组成一个团队的一组球员不仅取决于团队,还取决于正在考虑哪个游戏. 因此,我们不希望玩家与团队之间存在多对多关系. 我们也不希望玩家与游戏之间存在多对多关系. 
除了将玩家与任何这些模型相关联之外,我们需要的是玩家与 *团队－游戏约束* 之类的关联,因为这是一对(团队加游戏)来定义哪些玩家属于那里 .因此,我们正在寻找的正是联结模型GameTeam本身！并且,我们注意到,由于给定的 *游戏-团队* 指定了许多玩家,而同一位玩家可以参与许多 *游戏-团队*,因此我们需要玩家之间的多对多关系和GameTeam！

为了提供最大的灵活性,让我们在这里再次使用"超级多对多"关系构造：

```js
// Player 与 GameTeam 之间的超级多对多关系
const PlayerGameTeam = sequelize.define('PlayerGameTeam', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  }
});
Player.belongsToMany(GameTeam, { through: PlayerGameTeam });
GameTeam.belongsToMany(Player, { through: PlayerGameTeam });
PlayerGameTeam.belongsTo(Player);
PlayerGameTeam.belongsTo(GameTeam);
Player.hasMany(PlayerGameTeam);
GameTeam.hasMany(PlayerGameTeam);
```

上面的关联正是我们想要的. 这是一个完整的可运行示例：

```js
const { Sequelize, Op, Model, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:', {
  define: { timestamps: false } // 在这个例子中只是为了减少混乱
});
const Player = sequelize.define('Player', { username: DataTypes.STRING });
const Team = sequelize.define('Team', { name: DataTypes.STRING });
const Game = sequelize.define('Game', { name: DataTypes.INTEGER });

// 我们在 Game 和 Team 游戏和团队之间应用超级多对多关系
const GameTeam = sequelize.define('GameTeam', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  }
});
Team.belongsToMany(Game, { through: GameTeam });
Game.belongsToMany(Team, { through: GameTeam });
GameTeam.belongsTo(Game);
GameTeam.belongsTo(Team);
Game.hasMany(GameTeam);
Team.hasMany(GameTeam);

// 我们在 Player 和 GameTeam 游戏和团队之间应用超级多对多关系
const PlayerGameTeam = sequelize.define('PlayerGameTeam', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  }
});
Player.belongsToMany(GameTeam, { through: PlayerGameTeam });
GameTeam.belongsToMany(Player, { through: PlayerGameTeam });
PlayerGameTeam.belongsTo(Player);
PlayerGameTeam.belongsTo(GameTeam);
Player.hasMany(PlayerGameTeam);
GameTeam.hasMany(PlayerGameTeam);

(async () => {

  await sequelize.sync();
  await Player.bulkCreate([
    { username: 's0me0ne' },
    { username: 'empty' },
    { username: 'greenhead' },
    { username: 'not_spock' },
    { username: 'bowl_of_petunias' }
  ]);
  await Game.bulkCreate([
    { name: 'The Big Clash' },
    { name: 'Winter Showdown' },
    { name: 'Summer Beatdown' }
  ]);
  await Team.bulkCreate([
    { name: 'The Martians' },
    { name: 'The Earthlings' },
    { name: 'The Plutonians' }
  ]);

  // 让我们开始定义哪些球队参加了哪些比赛.
  // 这可以通过几种方式来完成,例如在每个游戏上调用`.setTeams`.
  // 但是,为简便起见,我们将直接使用 `create` 调用,
  // 直接引用我们想要的 ID. 我们知道 ID 是从 1 开始的.
  await GameTeam.bulkCreate([
    { GameId: 1, TeamId: 1 },   // 该 GameTeam 将获得 id 1
    { GameId: 1, TeamId: 2 },   // 该 GameTeam 将获得 id 2
    { GameId: 2, TeamId: 1 },   // 该 GameTeam 将获得 id 3
    { GameId: 2, TeamId: 3 },   // 该 GameTeam 将获得 id 4
    { GameId: 3, TeamId: 2 },   // 该 GameTeam 将获得 id 5
    { GameId: 3, TeamId: 3 }    // 该 GameTeam 将获得 id 6
  ]);

  // 现在让我们指定玩家.
  // 为简便起见,我们仅在第二场比赛(Winter Showdown)中这样做.
  // 比方说,s0me0ne 和 greenhead 效力于 Martians,
  // 而 not_spock 和 bowl_of_petunias 效力于 Plutonians:
  await PlayerGameTeam.bulkCreate([
    // 在 'Winter Showdown' (即 GameTeamIds 3 和 4)中:
    { PlayerId: 1, GameTeamId: 3 },   // s0me0ne played for The Martians
    { PlayerId: 3, GameTeamId: 3 },   // greenhead played for The Martians
    { PlayerId: 4, GameTeamId: 4 },   // not_spock played for The Plutonians
    { PlayerId: 5, GameTeamId: 4 }    // bowl_of_petunias played for The Plutonians
  ]);

  // 现在我们可以进行查询！
  const game = await Game.findOne({
    where: {
      name: "Winter Showdown"
    },
    include: {
      model: GameTeam,
      include: [
        {
          model: Player,
          through: { attributes: [] } // 隐藏结果中不需要的 `PlayerGameTeam` 嵌套对象
        },
        Team
      ]
    }
  });

  console.log(`Found game: "${game.name}"`);
  for (let i = 0; i < game.GameTeams.length; i++) {
    const team = game.GameTeams[i].Team;
    const players = game.GameTeams[i].Players;
    console.log(`- Team "${team.name}" played game "${game.name}" with the following players:`);
    console.log(players.map(p => `--- ${p.username}`).join('\n'));
  }

})();
```

输出:

```text
Found game: "Winter Showdown"
- Team "The Martians" played game "Winter Showdown" with the following players:
--- s0me0ne
--- greenhead
- Team "The Plutonians" played game "Winter Showdown" with the following players:
--- not_spock
--- bowl_of_petunias
```

因此,这就是我们利用超级多对多关系技术在 Sequelize 中实现三个模型之间的 *多对多对多* 关系的方式！

这个想法可以递归地应用于甚至更复杂的,*多对多对......对多* 关系(尽管有时查询可能会变慢).