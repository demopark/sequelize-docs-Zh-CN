# Upgrade to v7 - 升级到 V7

Sequelize v7 是 v6 之后的下一个主要版本. 以下列出了一些重大更改以帮助你进行升级.

## 重大变化

### 支持 Node 12 以及更高

Sequelize v7 将只支持那些与 ES 模块规范兼容的 Node.js 版本, 
版本 12 及更高版本[#5](https://github.com/sequelize/meetings/issues/5).

### TypeScript 转换

v7 的主要基础代码更改之一是迁移到 TypeScript.
结果就是, 以前在 JavaScript 代码库之上的尽力而为猜测的手动类型,
已被删除, 所有类型现在都直接从实际的 TypeScript 代码中检索。
您可能会发现许多微小的差异, 但它们应该很容易修复.

### 对 `ConnectionManager` 的更改

*仅当你直接使用 `ConnectionManager` 时, 这才会对你产生影响.*

`ConnectionManager#getConnection`: `type` 选项现在接受 `'read' | 'write'` 而不是 `'SELECT' | any`.
它已经在 v6 中记录, 但实现与文档不匹配.

```typescript
// 而不是这样做:
sequelize.connectionManager.getConnection({ type: 'SELECT' });

// 这样做:
sequelize.connectionManager.getConnection({ type: 'read' });
```

### Microsoft SQL Server 支持

Sequelize v7 完全支持 MS SQL Server 2017（版本 14）, Sequelize v6 从 2012（版本 13）开始
, 符合微软自己的[主流支持](
https://docs.microsoft.com/en-us/sql/sql-server/end-of-support/sql-server-end-of-life-overview?view=sql-server-ver15#lifecycle-dates).

### 不会在内部调用重写的模型方法

`Model.findOne` 和 `Model.findAll` 分别被 `Model.findByPk` 和 `Model.findOne` 使用.
这被认为是一个细节实现, 因此，从 Sequelize v7 开始，
`Model.findByPk` 或 `Model.findOne` 不会覆盖任何一个在这些方法中的内部调用。

换句话说, 这样做不会破坏:

```typescript
class User extends Model {
  static findOne() {
    throw new Error('Do not call findOne');
  }
}

// 这会在 v6 中抛出 "Do not call findOne"
// 但它在 v7 中有效
User.findByPk(1);
```
