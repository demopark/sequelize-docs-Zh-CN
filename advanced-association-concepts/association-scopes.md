# Association Scopes - 关联作用域

本节涉及关联作用域,它们与[模型作用域](other-topics/scopes.md)类似但不相同.

关联作用域既可以放在关联的模型(关联的目标)上,也可以放在多对多关系的联结表中.

## 概念

与[模型作用域](other-topics/scopes.md)如何自动应用于模型静态调用(例如 `Model.scope('foo').findAll()`)类似,关联作用域也是一个规则(更确切地说, 一组默认属性和参数),这些属性会自动应用于模型中的实例调用. 这里,*实例调用*是指从实例(而不是从 Model 本身)调用的方法调用. Mixins 是实例方法的主要示例(`instance.getSomething`, `instance.setSomething`, `instance.addSomething` 和 `instance.createSomething`).

关联作用域的行为就像模型作用域一样,在某种意义上,两者都导致将诸如 `where` 子句之类的内容自动应用到查找器调用; 区别在于,关联作用域不是自动应用于静态finder调用(模型范围就是这种情况),而是自动应用于实例finder调用(例如mixins).

## 示例

下面显示了模型 `Foo` 和 `Bar` 之间的一对多关联的关联范围的基本示例.

* 设置:

    ```js
    const Foo = sequelize.define('foo', { name: DataTypes.STRING });
    const Bar = sequelize.define('bar', { status: DataTypes.STRING });
    Foo.hasMany(Bar, {
        scope: {
            status: 'open'
        },
        as: 'openBars'
    });
    await sequelize.sync();
    const myFoo = await Foo.create({ name: "My Foo" });
    ```

* 设置完成后,调用 `myFoo.getOpenBars()` 会生成以下SQL：

    ```sql
    SELECT
        `id`, `status`, `createdAt`, `updatedAt`, `fooId`
    FROM `bars` AS `bar`
    WHERE `bar`.`status` = 'open' AND `bar`.`fooId` = 1;
    ```

这样,我们可以看到,调用 `.getOpenBars()` mixin 之后,关联作用域 `{ status: 'open' }` 被自动应用到生成的 SQL 的 `WHERE` 子句中.

## 在标准作用域内实现相同的行为

我们可以使用标准作用域实现相同的行为：

```js
// Foo.hasMany(Bar, {
//     scope: {
//         status: 'open'
//     },
//     as: 'openBars'
// });

Bar.addScope('open', {
    where: {
        status: 'open'
    }
});
Foo.hasMany(Bar);
Foo.hasMany(Bar.scope('open'), { as: 'openBars' });
```

使用上面的代码,`myFoo.getOpenBars()` 产生与上面所示相同的 SQL.