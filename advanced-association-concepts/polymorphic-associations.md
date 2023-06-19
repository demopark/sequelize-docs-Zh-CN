# Polymorphic Associations - 多态关联

_**注意:** 如本指南所述,在 Sequelize 中使用多态关联时应谨慎行事. 不要只是从此处复制粘贴代码,否则你可能会容易出错并在代码中引入错误. 请确保你了解发生了什么._

## 概念

一个 **多态关联** 由使用同一外键发生的两个(或多个)关联组成.

例如,考虑模型 `Image`, `Video` 和 `Comment`. 前两个代表用户可能发布的内容. 我们希望允许将评论放在两者中. 这样,我们立即想到建立以下关联：

* `Image` 和 `Comment` 之间的一对多关联:

  ```js
  Image.hasMany(Comment);
  Comment.belongsTo(Image);
  ```

* `Video` 和 `Comment` 之间的一对多关联:

  ```js
  Video.hasMany(Comment);
  Comment.belongsTo(Video);
  ```

但是,以上操作将导致 Sequelize 在 `Comment` 表上创建两个外键： `ImageId` 和 `VideoId`. 这是不理想的,因为这种结构使评论看起来可以同时附加到一个图像和一个视频上,这是不正确的. 取而代之的是,我们真正想要的是一个多态关联,其中一个 `Comment` 指向一个 **可评论**,它是表示 `Image` 或 `Video` 之一的抽象多态实体.

在继续配置此类关联之前,让我们看看如何使用它：

```js
const image = await Image.create({ url: "https://placekitten.com/408/287" });
const comment = await image.createComment({ content: "Awesome!" });

console.log(comment.commentableId === image.id); // true

// 我们还可以检索与评论关联的可评论类型.
// 下面显示了相关的可注释实例的模型名称.
console.log(comment.commentableType); // "Image"

// 我们可以使用多态方法来检索相关的可评论内容,
// 而不必关心它是图像还是视频.
const associatedCommentable = await comment.getCommentable();

// 在此示例中,`associatedCommentable` 与 `image` 是同一件事：
const isDeepEqual = require('deep-equal');
console.log(isDeepEqual(image, commentable)); // true
```

## 配置一对多多态关联

要为上述示例(这是一对多多态关联的示例)设置多态关联,我们需要执行以下步骤：

* 在 `Comment` 模型中定义一个名为 `commentableType` 的字符串字段;
* 在 `Image`/`Video` 和 `Comment` 之间定义 `hasMany` 和 `belongsTo` 关联:
  * 禁用约束(即使用 `{ constraints: false }`),因为同一个外键引用了多个表;
  * 指定适当的 [关联作用域](../advanced-association-concepts/association-scopes.md);
* 为了适当地支持延迟加载,请在 `Comment` 模型上定义一个名为 `getCommentable` 的新实例方法,该方法在后台调用正确的 mixin 来获取适当的注释对象;
* 为了正确支持预先加载,请在 `Comment` 模型上定义一个 `afterFind` hook,该 hook 将在每个实例中自动填充 `commentable` 字段;
* 为了防止预先加载的 bug/错误,你还可以在相同的 `afterFind` hook 中从 Comment 实例中删除具体字段 `image` 和 `video`,仅保留抽象的 `commentable` 字段可用.

这是一个示例:

```js
// Helper 方法
const uppercaseFirst = str => `${str[0].toUpperCase()}${str.substr(1)}`;

class Image extends Model {}
Image.init({
  title: DataTypes.STRING,
  url: DataTypes.STRING
}, { sequelize, modelName: 'image' });

class Video extends Model {}
Video.init({
  title: DataTypes.STRING,
  text: DataTypes.STRING
}, { sequelize, modelName: 'video' });

class Comment extends Model {
  getCommentable(options) {
    if (!this.commentableType) return Promise.resolve(null);
    const mixinMethodName = `get${uppercaseFirst(this.commentableType)}`;
    return this[mixinMethodName](options);
  }
}
Comment.init({
  title: DataTypes.STRING,
  commentableId: DataTypes.INTEGER,
  commentableType: DataTypes.STRING
}, { sequelize, modelName: 'comment' });

Image.hasMany(Comment, {
  foreignKey: 'commentableId',
  constraints: false,
  scope: {
    commentableType: 'image'
  }
});
Comment.belongsTo(Image, { foreignKey: 'commentableId', constraints: false });

Video.hasMany(Comment, {
  foreignKey: 'commentableId',
  constraints: false,
  scope: {
    commentableType: 'video'
  }
});
Comment.belongsTo(Video, { foreignKey: 'commentableId', constraints: false });

Comment.hooks.addListener("afterFind", findResult => {
  if (!Array.isArray(findResult)) findResult = [findResult];
  for (const instance of findResult) {
    if (instance.commentableType === "image" && instance.image !== undefined) {
      instance.commentable = instance.image;
    } else if (instance.commentableType === "video" && instance.video !== undefined) {
      instance.commentable = instance.video;
    }
    // 防止错误:
    delete instance.image;
    delete instance.dataValues.image;
    delete instance.video;
    delete instance.dataValues.video;
  }
});
```

由于 `commentableId` 列引用了多个表(本例中为两个表),因此我们无法向其添加 `REFERENCES` 约束. 这就是为什么使用 `constraints: false` 参数的原因.

注意,在上面的代码中:

* *Image -> Comment* 关联定义了一个关联作用域: `{ commentableType: 'image' }`
* *Video -> Comment* 关联定义了一个关联作用域: `{ commentableType: 'video' }`

使用关联函数时,这些作用域会自动应用(如[关联作用域](../advanced-association-concepts/association-scopes.md)指南中所述). 以下是一些示例及其生成的 SQL 语句：

* `image.getComments()`:

  ```sql
  SELECT "id", "title", "commentableType", "commentableId", "createdAt", "updatedAt"
  FROM "comments" AS "comment"
  WHERE "comment"."commentableType" = 'image' AND "comment"."commentableId" = 1;
  ```

在这里我们可以看到 `` `comment`.`commentableType` = 'image'`` 已自动添加到生成的 SQL 的 `WHERE` 子句中. 这正是我们想要的行为.

* `image.createComment({ title: 'Awesome!' })`:

  ```sql
  INSERT INTO "comments" (
    "id", "title", "commentableType", "commentableId", "createdAt", "updatedAt"
  ) VALUES (
    DEFAULT, 'Awesome!', 'image', 1,
    '2018-04-17 05:36:40.454 +00:00', '2018-04-17 05:36:40.454 +00:00'
  ) RETURNING *;
  ```

* `image.addComment(comment)`:

  ```sql
  UPDATE "comments"
  SET "commentableId"=1, "commentableType"='image', "updatedAt"='2018-04-17 05:38:43.948 +00:00'
  WHERE "id" IN (1)
  ```

### 多态延迟加载

`Comment` 上的 `getCommentable` 实例方法为延迟加载相关的 commentable 提供了一种抽象 - 无论注释属于 Image 还是 Video,都可以工作.

通过简单地将 `commentableType` 字符串转换为对正确的 mixin( `getImage` 或 `getVideo`)的调用即可工作.

注意上面的 `getCommentable` 实现：

* 不存在关联时返回 `null`;
* 允许你将参数对象传递给 `getCommentable(options)`,就像其他任何标准 Sequelize 方法一样. 对于示例,这对于指定 where 条件或 include 条件很有用.

### 多态预先加载

现在,我们希望对一个(或多个)注释执行关联的可评论对象的多态预先加载. 我们想要实现类似以下的东西：

```js
const comment = await Comment.findOne({
  include: [ /* ... */ ]
});
console.log(comment.commentable); // 这是我们的目标
```

解决的办法是告诉 Sequelize 同时包含图像和视频,以便上面定义的 `afterFind` hook可以完成工作,并自动向实例对象添加 `commentable` 字段,以提供所需的抽象.

示例:

```js
const comments = await Comment.findAll({
  include: [Image, Video]
});
for (const comment of comments) {
  const message = `Found comment #${comment.id} with ${comment.commentableType} commentable:`;
  console.log(message, comment.commentable.toJSON());
}
```

输出:

```text
Found comment #1 with image commentable: { id: 1,
  title: 'Meow',
  url: 'https://placekitten.com/408/287',
  createdAt: 2019-12-26T15:04:53.047Z,
  updatedAt: 2019-12-26T15:04:53.047Z }
```

### 注意 - 可能无效的 预先/延迟 加载!

注释 `Foo`,其 `commentableId` 为 2,而 `commentableType` 为 `image`. 然后 `Image A` 和 `Video X` 的 ID 都恰好等于 2.从概念上讲,很明显,`Video X` 与 `Foo` 没有关联,因为即使其 ID 为 2,`Foo` 的 `commentableType` 是 `image`,而不是 `video`. 然而,这种区分仅在 Sequelize 的 `getCommentable` 和我们在上面创建的 hook 执行的抽象级别上进行.

这意味着如果在上述情况下调用 `Comment.findAll({ include: Video })`,`Video X` 将被预先加载到 `Foo` 中. 幸运的是,我们的 `afterFind` hook将自动删除它,以帮助防止错误. 你了解发生了什么是非常重要的.

防止此类错误的最好方法是 **不惜一切代价直接使用具体的访问器和mixin** (例如 `.image`, `.getVideo()`, `.setImage()` 等) ,总是喜欢我们创建的抽象,例如 `.getCommentable()` and `.commentable`. 如果由于某种原因确实需要访问预先加载的 `.image` 和 `.video` 请确保将其包装在类型检查中,例如 `comment.commentableType === 'image'`.

## 配置多对多多态关联

在上面的示例中,我们将模型 `Image` 和 `Video` 抽象称为 *commentables*,其中一个 *commentable* 具有很多注释. 但是,一个给定的注释将属于一个 *commentable* - 这就是为什么整个情况都是一对多多态关联的原因.

现在,考虑多对多多态关联,而不是考虑注释,我们将考虑标签. 为了方便起见,我们现在将它们称为 *taggables*,而不是将它们称为 *commentables*. 一个 *taggable* 可以具有多个标签,同时一个标签可以放置在多个 *taggables* 中.

为此设置如下：

* 明确定义联结模型,将两个外键指定为 `tagId` 和 `taggableId`(这样,它是 `Tag` 与 *taggable* 抽象概念之间多对多关系的联结模型);
* 在联结模型中定义一个名为 `taggableType` 的字符串字段;
* 定义两个模型之间的 `belongsToMany` 关联和 `标签`:
  * 禁用约束 (即, 使用 `{ constraints: false }`), 因为同一个外键引用了多个表;
  * 指定适当的 [关联作用域](../advanced-association-concepts/association-scopes.md);
* 在 `Tag` 模型上定义一个名为 `getTaggables` 的新实例方法,该方法在后台调用正确的 mixin 来获取适当的 taggables.

实践:

```js
class Tag extends Model {
  getTaggables(options) {
    const images = await this.getImages(options);
    const videos = await this.getVideos(options);
    // 在单个 taggables 数组中合并 images 和 videos
    return images.concat(videos);
  }
}
Tag.init({
  name: DataTypes.STRING
}, { sequelize, modelName: 'tag' });

// 在这里,我们明确定义联结模型
class Tag_Taggable extends Model {}
Tag_Taggable.init({
  tagId: {
    type: DataTypes.INTEGER,
    unique: 'tt_unique_constraint'
  },
  taggableId: {
    type: DataTypes.INTEGER,
    unique: 'tt_unique_constraint',
    references: null
  },
  taggableType: {
    type: DataTypes.STRING,
    unique: 'tt_unique_constraint'
  }
}, { sequelize, modelName: 'tag_taggable' });

Image.belongsToMany(Tag, {
  through: {
    model: Tag_Taggable,
    unique: false,
    scope: {
      taggableType: 'image'
    }
  },
  foreignKey: 'taggableId',
  constraints: false
});
Tag.belongsToMany(Image, {
  through: {
    model: Tag_Taggable,
    unique: false
  },
  foreignKey: 'tagId',
  constraints: false
});

Video.belongsToMany(Tag, {
  through: {
    model: Tag_Taggable,
    unique: false,
    scope: {
      taggableType: 'video'
    }
  },
  foreignKey: 'taggableId',
  constraints: false
});
Tag.belongsToMany(Video, {
  through: {
    model: Tag_Taggable,
    unique: false
  },
  foreignKey: 'tagId',
  constraints: false
});
```

`constraints: false`  参数禁用引用约束,因为 `taggableId` 列引用了多个表,因此我们无法向其添加 `REFERENCES` 约束.

注意下面:

* 对 *Image -> Tag* 关联定义了一个关联范围: `{ taggableType: 'image' }`
* 对 *Video -> Tag* 关联定义了一个关联范围: `{ taggableType: 'video' }`

使用关联函数时,将自动应用这些作用域. 以下是一些示例及其生成的 SQL 语句：

* `image.getTags()`:

  ```sql
  SELECT
    `tag`.`id`,
    `tag`.`name`,
    `tag`.`createdAt`,
    `tag`.`updatedAt`,
    `tag_taggable`.`tagId` AS `tag_taggable.tagId`,
    `tag_taggable`.`taggableId` AS `tag_taggable.taggableId`,
    `tag_taggable`.`taggableType` AS `tag_taggable.taggableType`,
    `tag_taggable`.`createdAt` AS `tag_taggable.createdAt`,
    `tag_taggable`.`updatedAt` AS `tag_taggable.updatedAt`
  FROM `tags` AS `tag`
  INNER JOIN `tag_taggables` AS `tag_taggable` ON
    `tag`.`id` = `tag_taggable`.`tagId` AND
    `tag_taggable`.`taggableId` = 1 AND
    `tag_taggable`.`taggableType` = 'image';
  ```

在这里我们可以看到 `` `tag_taggable`.`taggableType` = 'image'`` 已被自动添加到生成的 SQL 的 WHERE 子句中. 这正是我们想要的行为.

* `tag.getTaggables()`:

  ```sql
  SELECT
    `image`.`id`,
    `image`.`url`,
    `image`.`createdAt`,
    `image`.`updatedAt`,
    `tag_taggable`.`tagId` AS `tag_taggable.tagId`,
    `tag_taggable`.`taggableId` AS `tag_taggable.taggableId`,
    `tag_taggable`.`taggableType` AS `tag_taggable.taggableType`,
    `tag_taggable`.`createdAt` AS `tag_taggable.createdAt`,
    `tag_taggable`.`updatedAt` AS `tag_taggable.updatedAt`
  FROM `images` AS `image`
  INNER JOIN `tag_taggables` AS `tag_taggable` ON
    `image`.`id` = `tag_taggable`.`taggableId` AND
    `tag_taggable`.`tagId` = 1;
  
  SELECT
    `video`.`id`,
    `video`.`url`,
    `video`.`createdAt`,
    `video`.`updatedAt`,
    `tag_taggable`.`tagId` AS `tag_taggable.tagId`,
    `tag_taggable`.`taggableId` AS `tag_taggable.taggableId`,
    `tag_taggable`.`taggableType` AS `tag_taggable.taggableType`,
    `tag_taggable`.`createdAt` AS `tag_taggable.createdAt`,
    `tag_taggable`.`updatedAt` AS `tag_taggable.updatedAt`
  FROM `videos` AS `video`
  INNER JOIN `tag_taggables` AS `tag_taggable` ON
    `video`.`id` = `tag_taggable`.`taggableId` AND
    `tag_taggable`.`tagId` = 1;
  ```

请注意,上述 `getTaggables()` 的实现允许你将选项对象传递给 `getCommentable(options)`,就像其他任何标准 Sequelize 方法一样. 例如,这对于指定条件或包含条件很有用.

### 在目标模型上应用作用域

在上面的示例中,`scope` 参数(例如 `scope: { taggableType: 'image' }`)应用于 *联结* 模型,而不是 *目标* 模型,因为它是在 `through` 下使用的参数.

我们还可以在目标模型上应用关联作用域. 我们甚至可以同时进行.

为了说明这一点,请考虑上述示例在标签和可标记之间的扩展,其中每个标签都有一个状态. 这样,为了获取图像的所有待处理标签,我们可以在 `Image` 和 `Tag` 之间建立另一个 `belognsToMany` 关系,这一次在联结模型上应用作用域,在目标模型上应用另一个作用域：

```js
Image.belongsToMany(Tag, {
  through: {
    model: Tag_Taggable,
    unique: false,
    scope: {
      taggableType: 'image'
    }
  },
  scope: {
    status: 'pending'
  },
  as: 'pendingTags',
  foreignKey: 'taggableId',
  constraints: false
});
```

这样,当调用 `image.getPendingTags()` 时,将生成以下 SQL 查询：

```sql
SELECT
  `tag`.`id`,
  `tag`.`name`,
  `tag`.`status`,
  `tag`.`createdAt`,
  `tag`.`updatedAt`,
  `tag_taggable`.`tagId` AS `tag_taggable.tagId`,
  `tag_taggable`.`taggableId` AS `tag_taggable.taggableId`,
  `tag_taggable`.`taggableType` AS `tag_taggable.taggableType`,
  `tag_taggable`.`createdAt` AS `tag_taggable.createdAt`,
  `tag_taggable`.`updatedAt` AS `tag_taggable.updatedAt`
FROM `tags` AS `tag`
INNER JOIN `tag_taggables` AS `tag_taggable` ON
  `tag`.`id` = `tag_taggable`.`tagId` AND
  `tag_taggable`.`taggableId` = 1 AND
  `tag_taggable`.`taggableType` = 'image'
WHERE (
  `tag`.`status` = 'pending'
);
```

我们可以看到两个作用域都是自动应用的:

* `` `tag_taggable`.`taggableType` = 'image'`` 被自动添加到 `INNER JOIN`;
* `` `tag`.`status` = 'pending'`` 被自动添加到外部 where 子句.