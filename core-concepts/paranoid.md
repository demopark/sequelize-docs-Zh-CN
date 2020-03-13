# Paranoid - 偏执表

Sequelize 支持 *paranoid* 表的概念. 一个 *paranoid* 表是一个被告知删除记录时不会真正删除它的表.反而一个名为 `deletedAt` 的特殊列会将其值设置为该删除请求的时间戳.

这意味着偏执表会执行记录的 *软删除*,而不是 *硬删除*.

## 将模型定义为 paranoid

要定义 paranoid 模型,必须将 `paranoid: true` 参数传递给模型定义. Paranoid 需要时间戳才能起作用(即,如果你传递 `timestamps: false` 了,paranoid 将不起作用).

你还可以将默认的列名(默认是 `deletedAt`)更改为其他名称.

```js
class Post extends Model {}
Post.init({ /* 这是属性 */ }, {
  sequelize,
  paranoid: true,

  // 如果要为 deletedAt 列指定自定义名称
  deletedAt: 'destroyTime'
});
```

## 删除

当你调用 `destroy` 方法时,将发生软删除：

```js
await Post.destroy({
  where: {
    id: 1
  }
});
// UPDATE "posts" SET "deletedAt"=[timestamp] WHERE "deletedAt" IS NULL AND "id" = 1
```

如果你确实想要硬删除,并且模型是 paranoid,则可以使用 `force: true` 参数强制执行：

```js
await Post.destroy({
  where: {
    id: 1
  },
  force: true
});
// DELETE FROM "posts" WHERE "id" = 1
```

上面的示例以静态的 `destroy` 方法为例(`Post.destroy`),所有实例方法的工作方式相同：

```js
const post = await Post.create({ title: 'test' });
console.log(post instanceof Post); // true
await post.destroy(); // 只设置 `deletedAt` 标志
await post.destroy({ force: true }); // 真的会删除记录
```

## 恢复

要恢复软删除的记录,可以使用 `restore` 方法,该方法在静态版本和实例版本中都提供：

```js
// 展示实例 `restore` 方法的示例
// 我们创建一个帖子,对其进行软删除,然后将其还原
const post = await Post.create({ title: 'test' });
console.log(post instanceof Post); // true
await post.destroy();
console.log('soft-deleted!');
await post.restore();
console.log('restored!');

// 展示静态 `restore` 方法的示例.
// 恢复每个 likes 大于 100 的软删除的帖子
await Post.restore({
  where: {
    likes: {
      [Op.gt]: 100
    }
  }
});
```

## 其他查询行为

Sequelize 执行的每个查询将自动忽略软删除的记录(当然,原始查询除外).

这意味着,例如,`findAll` 方法将看不到软删除的记录,仅获取未删除的记录.

即使你单纯的调用提供了软删除记录主键的`findByPk`,结果也将是 `null`,就好像该记录不存在一样.

如果你真的想让查询看到被软删除的记录,可以将 `paranoid: false` 参数传递给查询方法. 例如：

```js
await Post.findByPk(123); // 如果 ID 123 的记录被软删除,则将返回 `null`
await Post.findByPk(123, { paranoid: false }); // 这将检索记录

await Post.findAll({
  where: { foo: 'bar' }
}); // 这将不会检索软删除的记录

await Post.findAll({
  where: { foo: 'bar' },
  paranoid: false
}); // 这还将检索软删除的记录
```