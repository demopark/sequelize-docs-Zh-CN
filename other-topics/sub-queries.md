# Sub Queries - 子查询

考虑你有两个模型,即 `Post` 和 `Reaction`,它们之间建立了一对多的关系,因此一个 post 有很多 reactions：

```js
const Post = sequelize.define('post', {
    content: DataTypes.STRING
}, { timestamps: false });

const Reaction = sequelize.define('reaction', {
    type: DataTypes.STRING
}, { timestamps: false });

Post.hasMany(Reaction);
Reaction.belongsTo(Post);
```

*注意: 我们已禁用时间戳,只是为了缩短下一个示例的查询时间.*

让我们用一些数据填充表格:

```js
async function makePostWithReactions(content, reactionTypes) {
    const post = await Post.create({ content });
    await Reaction.bulkCreate(
        reactionTypes.map(type => ({ type, postId: post.id }))
    );
    return post;
}

await makePostWithReactions('Hello World', [
    'Like', 'Angry', 'Laugh', 'Like', 'Like', 'Angry', 'Sad', 'Like'
]);
await makePostWithReactions('My Second Post', [
    'Laugh', 'Laugh', 'Like', 'Laugh'
]);
```

现在,我们已经准备好子查询功能的示例.

假设我们要通过 SQL 为每个帖子计算一个 `laughReactionsCount `. 我们可以通过子查询来实现,例如：

```sql
SELECT
    *,
    (
        SELECT COUNT(*)
        FROM reactions AS reaction
        WHERE
            reaction.postId = post.id
            AND
            reaction.type = "Laugh"
    ) AS laughReactionsCount
FROM posts AS post
```

如果我们通过 Sequelize 运行上面的原始 SQL 查询,我们将得到:

```json
[
  {
    "id": 1,
    "content": "Hello World",
    "laughReactionsCount": 1
  },
  {
    "id": 2,
    "content": "My Second Post",
    "laughReactionsCount": 3
  }
]
```

那么,如何在 Sequelize 的帮助下实现这一目标,而不必手工编写整个原始查询呢？

答案是: 通过将 finder 方法(例如,`findAll `)的 `attributes ` 参数与 `sequelize.literal` 实用程序功能结合使用,可以直接在查询中插入任意内容,而不会自动转义.

这意味着 Sequelize 将帮助你进行较大的主要查询,但是你仍然必须自己编写该子查询：

```js
Post.findAll({
    attributes: {
        include: [
            [
                // 注意下面的调用中的括号！
                sequelize.literal(`(
                    SELECT COUNT(*)
                    FROM reactions AS reaction
                    WHERE
                        reaction.postId = post.id
                        AND
                        reaction.type = "Laugh"
                )`),
                'laughReactionsCount'
            ]
        ]
    }
});
```

*重要提示：由于 `sequelize.literal` 会插入任意内容而不进行转义,因此,它可能是(主要)安全漏洞的来源,因此值得特别注意. 它不应该在用户生成的内容上使用.*但是在这里,我们使用自己编写的带有固定字符串的 `sequelize.literal`.因为我们知道我们在做什么.

上面给出了以下输出:

```json
[
  {
    "id": 1,
    "content": "Hello World",
    "laughReactionsCount": 1
  },
  {
    "id": 2,
    "content": "My Second Post",
    "laughReactionsCount": 3
  }
]
```

成功!

## 使用子查询进行复杂排序

这个想法可用于实现复杂的排序,例如根据 post 具有的 laugh 数量来排序帖子：

```js
Post.findAll({
    attributes: {
        include: [
            [
                sequelize.literal(`(
                    SELECT COUNT(*)
                    FROM reactions AS reaction
                    WHERE
                        reaction.postId = post.id
                        AND
                        reaction.type = "Laugh"
                )`),
                'laughReactionsCount'
            ]
        ]
    },
    order: [
        [sequelize.literal('laughReactionsCount'), 'DESC']
    ]
});
```

结果:

```json
[
  {
    "id": 2,
    "content": "My Second Post",
    "laughReactionsCount": 3
  },
  {
    "id": 1,
    "content": "Hello World",
    "laughReactionsCount": 1
  }
]
```