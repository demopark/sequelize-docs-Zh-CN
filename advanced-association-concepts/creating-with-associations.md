# Creating with Associations - 创建关联

只要所有元素都是新元素,就可以一步创建带有嵌套关联的实例.

相反,无法执行涉及嵌套对象的更新和删除. 为此,你将必须明确执行每个单独的操作.

## BelongsTo / HasMany / HasOne 关联

考虑以下模型:

```js
class Product extends Model {}
Product.init({
  title: DataTypes.STRING
}, { sequelize, modelName: 'product' });
class User extends Model {}
User.init({
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING
}, { sequelize, modelName: 'user' });
class Address extends Model {}
Address.init({
  type: DataTypes.STRING,
  line1: DataTypes.STRING,
  line2: DataTypes.STRING,
  city: DataTypes.STRING,
  state: DataTypes.STRING,
  zip: DataTypes.STRING,
}, { sequelize, modelName: 'address' });

// 我们保存关联设置调用的返回值,以便以后使用
Product.User = Product.belongsTo(User);
User.Addresses = User.hasMany(Address);
// 也适用于 `hasOne`
```

一个新的 `Product`,`User` 和一个或多个 `Address` 可以按以下步骤一步创建：

```js
return Product.create({
  title: 'Chair',
  user: {
    firstName: 'Mick',
    lastName: 'Broadstone',
    addresses: [{
      type: 'home',
      line1: '100 Main St.',
      city: 'Austin',
      state: 'TX',
      zip: '78704'
    }]
  }
}, {
  include: [{
    association: Product.User,
    include: [ User.Addresses ]
  }]
});
```

观察 `Product.create` 调用中 `include` 参数的用法. 这对于 Sequelize 理解与关联一起创建的内容很有必要.

注意：这里,我们的用户模型称为`user`,小写的`u`-这意味着对象中的属性也应为`user`. 如果给`sequelize.define`的名称是`User`,则对象中的 key 也应该是`User`. 对于 `addresses` 也是如此,除了它是 `hasMany` 关联的复数形式.

## 一个别名 BelongsTo 关联

可以扩展前面的示例以支持关联别名.

```js
const Creator = Product.belongsTo(User, { as: 'creator' });

return Product.create({
  title: 'Chair',
  creator: {
    firstName: 'Matt',
    lastName: 'Hansen'
  }
}, {
  include: [ Creator ]
});
```

## HasMany / BelongsToMany 关联

让我们介绍将产品与许多标签关联的功能. 设置模型如下所示：

```js
class Tag extends Model {}
Tag.init({
  name: DataTypes.STRING
}, { sequelize, modelName: 'tag' });

Product.hasMany(Tag);
// 也适用于 `belongsToMany`.
```

现在,我们可以通过以下方式创建具有多个标签的产品：

```js
Product.create({
  id: 1,
  title: 'Chair',
  tags: [
    { name: 'Alpha'},
    { name: 'Beta'}
  ]
}, {
  include: [ Tag ]
})
```

并且,我们可以修改此示例以支持别名：

```js
const Categories = Product.hasMany(Tag, { as: 'categories' });

Product.create({
  id: 1,
  title: 'Chair',
  categories: [
    { id: 1, name: 'Alpha' },
    { id: 2, name: 'Beta' }
  ]
}, {
  include: [{
    association: Categories,
    as: 'categories'
  }]
})
```