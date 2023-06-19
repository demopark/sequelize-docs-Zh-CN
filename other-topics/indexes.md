# Indexes - 索引

Sequelize 支持在模型定义上添加索引,该索引将在 [`sequelize.sync()`](/api/v7/classes/Sequelize.html#sync) 上创建

```js
const User = sequelize.define('User', { /* 属性 */ }, {
  indexes: [
    // 在 email 上创建唯一索引
    {
      unique: true,
      fields: ['email']
    },

    // 使用 jsonb_path_ops 运算符在 data 上创建 gin 索引
    {
      fields: ['data'],
      using: 'gin',
      operator: 'jsonb_path_ops'
    },

    // 默认情况下,索引名称将为 [table]_[fields]
    // 创建多列部分索引
    {
      name: 'public_by_author',
      fields: ['author', 'status'],
      where: {
        status: 'public'
      }
    },

    // 具有 order 字段的 BTREE 索引
    {
      name: 'title_index',
      using: 'BTREE',
      fields: [
        'author',
        {
          name: 'title',
          collate: 'en_US',
          order: 'DESC',
          length: 5
        }
      ]
    }
  ]
});
```