# Working with legacy tables - 使用遗留表

虽然 Sequelize 自认为可以开箱即用, 但是如果你要使用应用之前遗留的资产和凭据,仅需要做一点微不足道的设置即可。

## 表
```js
sequelize.define('user', {

}, {
  tableName: 'users'
});
```

## 字段
```js
sequelize.define('modelName', {
  userId: {
    type: Sequelize.INTEGER,
    field: 'user_id'
  }
});
```

## 主键

Sequelize将假设您的表默认具有`id`主键属性。

要定义你自己的主键：

```js
sequelize.define('collection', {
  uid: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true // Automatically gets converted to SERIAL for postgres
  }
});

sequelize.define('collection', {
  uuid: {
    type: Sequelize.UUID,
    primaryKey: true
  }
});
```

如果你的模型根本没有主键，你可以使用 `Model.removeAttribute('id');`

## 外键
```js
// 1:1
Organization.belongsTo(User, {foreignKey: 'owner_id'});
User.hasOne(Organization, {foreignKey: 'owner_id'});

// 1:M
Project.hasMany(Task, {foreignKey: 'tasks_pk'});
Task.belongsTo(Project, {foreignKey: 'tasks_pk'});

// N:M
User.hasMany(Role, {through: 'user_has_roles', foreignKey: 'user_role_user_id'});
Role.hasMany(User, {through: 'user_has_roles', foreignKey: 'roles_identifier'});
```
