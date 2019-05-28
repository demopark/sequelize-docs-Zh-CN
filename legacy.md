# Working with legacy tables - 使用遗留表

虽然 Sequelize 自认为可以开箱即用, 但是如果你要使用应用之前遗留的资产和凭据,仅需要通过定义(否则生成)表和字段名称即可.

## 表
```js
class User extends Model {}
User.init({
  // ...
}, {
  modelName: 'user',
  tableName: 'users',
  sequelize,
});
```

## 字段
```js
class MyModel extends Model {}
MyModel.init({
  userId: {
    type: Sequelize.INTEGER,
    field: 'user_id'
  }
}, { sequelize });
```

## 主键

Sequelize将假设你的表默认具有`id`主键属性.

要定义你自己的主键:

```js
class Collection extends Model {}
Collection.init({
  uid: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true // 对于 postgres 自动转换为 SERIAL
  }
}, { sequelize });

class Collection extends Model {}
Collection.init({
  uuid: {
    type: Sequelize.UUID,
    primaryKey: true
  }
}, { sequelize });
```

如果你的模型根本没有主键,你可以使用 `Model.removeAttribute('id');`

## 外键
```js
// 1:1
Organization.belongsTo(User, { foreignKey: 'owner_id' });
User.hasOne(Organization, { foreignKey: 'owner_id' });

// 1:M
Project.hasMany(Task, { foreignKey: 'tasks_pk' });
Task.belongsTo(Project, { foreignKey: 'tasks_pk' });

// N:M
User.belongsToMany(Role, { through: 'user_has_roles', foreignKey: 'user_role_user_id' });
Role.belongsToMany(User, { through: 'user_has_roles', foreignKey: 'roles_identifier' });
```
