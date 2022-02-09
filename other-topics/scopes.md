# Scopes - 作用域

作用域用于帮助你重用代码. 你可以定义常用查询,并指定诸如 `where`, `include`, `limit` 等参数.

本指南涉及模型作用域. 你可能也对[关联作用域指南](../advanced-association-concepts/association-scopes.md)感兴趣,它们相似但又不同.

## 定义

作用域在模型定义中定义,可以是查找器对象,也可以是返回查找器对象的函数 - 默认作用域除外,该作用域只能是一个对象：

```js
class Project extends Model {}
Project.init({
  // 属性
}, {
  defaultScope: {
    where: {
      active: true
    }
  },
  scopes: {
    deleted: {
      where: {
        deleted: true
      }
    },
    activeUsers: {
      include: [
        { model: User, where: { active: true } }
      ]
    },
    random() {
      return {
        where: {
          someNumber: Math.random()
        }
      }
    },
    accessLevel(value) {
      return {
        where: {
          accessLevel: {
            [Op.gte]: value
          }
        }
      }
    },
    sequelize,
    modelName: 'project'
  }
});
```

你也可以在定义模型后通过调用 [`YourModel.addScope`](https://sequelize.org/master/class/lib/model.js~Model.html#static-method-addScope) 添加作用域. 这对于具有包含的作用域特别有用,其中在定义另一个模型时可能未定义包含中的模型.

始终应用默认作用域. 这意味着,使用上面的模型定义,`Project.findAll()` 将创建以下查询：

```sql
SELECT * FROM projects WHERE active = true
```

可以通过调用 `.unscoped()`, `.scope(null)`, 或调用另一个作用域来删除默认作用域：

```js
await Project.scope('deleted').findAll(); // 删除默认作用域
```

```sql
SELECT * FROM projects WHERE deleted = true
```

也可以在作用域定义中包括作用域模型. 这样可以避免重复 `include`, `attributes` 或 `where` 定义. 使用上面的示例,并在包含的用户模型上调用  `active` 作用域(而不是直接在该包含对象中指定条件)：

```js
// 上例中定义的 `activeUsers` 作用域也可以通过以下方式定义：
Project.addScope('activeUsers', {
  include: [
    { model: User.scope('active') }
  ]
});
```

## 使用

通过在模型定义上调用 `.scope`,并传递一个或多个作用域的名称来应用作用域.`.scope` 返回具有所有常规方法的功能齐全的模型实例：`.findAll`, `.update`, `.count`, `.destroy` 等.你可以保存此模型实例并在以后重用：

```js
const DeletedProjects = Project.scope('deleted');
await DeletedProjects.findAll();

// 以上相当于:
await Project.findAll({
  where: {
    deleted: true
  }
});
```

作用域适用于 `.find`, `.findAll`, `.count`, `.update`, `.increment` 和 `.destroy`.

作用域可以通过两种方式调用. 如果作用域不带任何参数,则可以正常调用它. 如果作用域接受参数,则传递一个对象：

```js
await Project.scope('random', { method: ['accessLevel', 19] }).findAll();
```

生成 SQL:

```sql
SELECT * FROM projects WHERE someNumber = 42 AND accessLevel >= 19
```

## 合并

通过将作用域数组传递给 `.scope` 或将作用域作为连续参数传递,可以同时应用多个作用域.

```js
// 这两个是等效的
await Project.scope('deleted', 'activeUsers').findAll();
await Project.scope(['deleted', 'activeUsers']).findAll();
```

生成 SQL:

```sql
SELECT * FROM projects
INNER JOIN users ON projects.userId = users.id
WHERE projects.deleted = true
AND users.active = true
```

如果要在默认作用域之外应用另一个作用域,请将关键字 `defaultScope` 传递给`.scope`：

```js
await Project.scope('defaultScope', 'deleted').findAll();
```

生成 SQL:

```sql
SELECT * FROM projects WHERE active = true AND deleted = true
```

调用多个合并作用域时,后续合并作用域中的键将覆盖先前合并作用域中的键(类似于 [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)),除了将合并的 `where` 和 `include` 之外. 考虑两个作用域：

```js
YourMode.addScope('scope1', {
  where: {
    firstName: 'bob',
    age: {
      [Op.gt]: 20
    }
  },
  limit: 2
});
YourMode.addScope('scope2', {
  where: {
    age: {
      [Op.gt]: 30
    }
  },
  limit: 10
});
```

使用 `.scope('scope1', 'scope2')` 将产生以下 WHERE 子句：

```sql
WHERE firstName = 'bob' AND age > 30 LIMIT 10
```

注意 `limit` 和 `age` 如何被 `scope2` 覆盖,而保留 `firstName`.`limit`, `offset`, `order`, `paranoid`, `lock` 和 `raw` 字段被覆盖,而 `where` 则被浅合并(这意味着相同的键将被覆盖).包含的合并策略将在后面讨论.

注意,多个应用作用域的 `attributes` 键以始终保留 `attributes.exclude` 的方式合并. 这允许合并多个合并作用域,并且永远不会泄漏最终合并作用域中的敏感字段.

当将查找对象直接传递给作用域模型上的 `findAll`(和类似的查找器)时,适用相同的合并逻辑：

```js
Project.scope('deleted').findAll({
  where: {
    firstName: 'john'
  }
})
```

生成的 where 子句：

```sql
WHERE deleted = true AND firstName = 'john'
```

在这里, `deleted` 作用域与查找器合并. 如果我们将 `where: { firstName: 'john', deleted: false }` 传递给查找器,则 `deleted` 作用域将被覆盖.

### 合并 Include

Include 将基于所包含的模型进行递归合并. 这是 v5 上添加的功能非常强大的合并,并通过示例更好地理解.

考虑模型 `Foo`, `Bar`, `Baz` 和 `Qux`,它们具有一对多关联,如下所示：

```js
const Foo = sequelize.define('Foo', { name: Sequelize.STRING });
const Bar = sequelize.define('Bar', { name: Sequelize.STRING });
const Baz = sequelize.define('Baz', { name: Sequelize.STRING });
const Qux = sequelize.define('Qux', { name: Sequelize.STRING });
Foo.hasMany(Bar, { foreignKey: 'fooId' });
Bar.hasMany(Baz, { foreignKey: 'barId' });
Baz.hasMany(Qux, { foreignKey: 'bazId' });
```

现在,考虑在 Foo 上定义的以下四个作用域:

```js
Foo.addScope('includeEverything', {
  include: {
    model: Bar,
    include: [{
      model: Baz,
      include: Qux
    }]
  }
});

Foo.addScope('limitedBars', {
  include: [{
    model: Bar,
    limit: 2
  }]
});

Foo.addScope('limitedBazs', {
  include: [{
    model: Bar,
    include: [{
      model: Baz,
      limit: 2
    }]
  }]
});

Foo.addScope('excludeBazName', {
  include: [{
    model: Bar,
    include: [{
      model: Baz,
      attributes: {
        exclude: ['name']
      }
    }]
  }]
});
```

这四个作用域可以很容易地进行深度合并,例如,通过调用 `Foo.scope('includeEverything', 'limitedBars', 'limitedBazs', 'excludeBazName').findAll()`,这完全等同于调用以下代码：

```js
await Foo.findAll({
  include: {
    model: Bar,
    limit: 2,
    include: [{
      model: Baz,
      limit: 2,
      attributes: {
        exclude: ['name']
      },
      include: Qux
    }]
  }
});

// 以上等同于：
await Foo.scope([
  'includeEverything',
  'limitedBars',
  'limitedBazs',
  'excludeBazName'
]).findAll();
```

观察如何将四个作用域合并为一个. 作用域的 include 根据所包含的模型进行合并. 如果一个作用域包含模型 A,另一个作用域包含模型 B,则合并结果将同时包含模型 A 和 B.另一方面,如果两个作用域都包含相同的模型 A,但具有不同的参数(例如嵌套包含或其他属性) ,这些将被递归合并,如上所示.

上面说明的合并以完全相同的方式工作,而不管应用于作用域的顺序如何. 如果某个选项由两个不同的作用域设置,则顺序只会有所不同 - 上面的示例不是这种情况,因为每个作用域执行的操作都不相同.

这种合并策略还可以与传递给 `.findAll`, `.findOne` 等的参数完全相同.