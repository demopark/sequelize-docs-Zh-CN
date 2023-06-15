# Working with Legacy Tables - 使用遗留表

虽然 Sequelize 自认为可以开箱即用, 但是如果你要处理遗留表并向前验证应用程序,仅需要通过定义(否则生成)表和字段名称即可.

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
    type: DataTypes.INTEGER,
    field: 'user_id'
  }
}, { sequelize });
```

## 主键

默认情况下,Sequelize 会假设你的表具有 `id` 主键属性.

定义自己的主键:

```js
class Collection extends Model {}
Collection.init({
  uid: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true // 自动转换为 PostgreSQL 的 SERIAL
  }
}, { sequelize });

class Collection extends Model {}
Collection.init({
  uuid: {
    type: DataTypes.UUID,
    primaryKey: true
  }
}, { sequelize });
```

如果你的模型根本没有主键,则可以使用 `Model.removeAttribute('id');`

仍然可以使用 `Model.findOne` 和 `Model.findAll` 检索没有主键的实例.
虽然目前可以使用它们的实例方法 (`instance.save`、`instance.update` 等). 但这样做会导致细微的错误, 并且计划在未来的主要更新中删除.

**注意**

如果你的模型没有主键, 则需要使用以下实例方法的静态等效项, 并提供你自己的 `where` 参数:

- `instance.save`: `Model.update`
- `instance.update`: `Model.update`
- `instance.reload`: `Model.findOne`
- `instance.destroy`: `Model.destroy`
- `instance.restore`: `Model.restore`
- `instance.decrement`: `Model.decrement`
- `instance.increment`: `Model.increment`

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
