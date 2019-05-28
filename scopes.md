# Scopes - 作用域

作用域允许你定义常用查询,以便以后轻松使用. 作用域可以包括与常规查找器 `where`, `include`, `limit` 等所有相同的属性.

## 定义

作用域在模型定义中定义,可以是finder对象或返回finder对象的函数,除了默认作用域,该作用域只能是一个对象:

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
        { model: User, where: { active: true }}
      ]
    },
    random () {
      return {
        where: {
          someNumber: Math.random()
        }
      }
    },
    accessLevel (value) {
      return {
        where: {
          accessLevel: {
            [Op.gte]: value
          }
        }
      }
    }
    sequelize,
    modelName: 'project'
  }
});
```

通过调用 `addScope` 定义模型后,还可以添加作用域. 这对于具有包含的作用域特别有用,其中在定义其他模型时可能不会定义 include 中的模型.

始终应用默认作用域. 这意味着,通过上面的模型定义,`Project.findAll()` 将创建以下查询:

```sql
SELECT * FROM projects WHERE active = true
```

可以通过调用 `.unscoped()`, `.scope(null)` 或通过调用另一个作用域来删除默认作用域:

```js
Project.scope('deleted').findAll(); // 删除默认作用域
```
```sql
SELECT * FROM projects WHERE deleted = true
```

还可以在作用域定义中包含作用域模型. 这让你避免重复 `include`,`attributes` 或 `where` 定义.

使用上面的例子,并在包含的用户模型中调用 `active` 作用域(而不是直接在该 include 对象中指定条件):

```js
activeUsers: {
  include: [
    { model: User.scope('active')}
  ]
}
```

## 使用

通过在模型定义上调用 `.scope` 来应用作用域,传递一个或多个作用域的名称. `.scope` 返回一个全功能的模型实例,它具有所有常规的方法:`.findAll`,`.update`,`.count`,`.destroy`等等.你可以保存这个模型实例并稍后再次使用:

```js
const DeletedProjects = Project.scope('deleted');

DeletedProjects.findAll();
// 过一段时间

// 让我们再次寻找被删除的项目！
DeletedProjects.findAll();
```

作用域适用于  `.find`, `.findAll`, `.count`, `.update`, `.increment` 和 `.destroy`.

可以通过两种方式调用作为函数的作用域. 如果作用域没有任何参数,它可以正常调用. 如果作用域采用参数,则传递一个对象:

```js
Project.scope('random', { method: ['accessLevel', 19]}).findAll();
```

```sql
SELECT * FROM projects WHERE someNumber = 42 AND accessLevel >= 19
```

## 合并

通过将作用域数组传递到 `.scope` 或通过将作用域作为连续参数传递,可以同时应用多个作用域.

```js
// 这两个是等价的
Project.scope('deleted', 'activeUsers').findAll();
Project.scope(['deleted', 'activeUsers']).findAll();
```

```sql
SELECT * FROM projects
INNER JOIN users ON projects.userId = users.id
WHERE projects.deleted = true
AND users.active = true
```

如果要将其他作用域与默认作用域一起应用,请将键 `defaultScope` 传递给 `.scope`:

```js
Project.scope('defaultScope', 'deleted').findAll();
```

```sql
SELECT * FROM projects WHERE active = true AND deleted = true
```

当调用多个作用域时,后续作用域的键将覆盖以前的作用域(类似于  [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)),除了`where`和`include`,它们将被合并. 考虑两个作用域:

```js
{
  scope1: {
    where: {
      firstName: 'bob',
      age: {
        [Op.gt]: 20
      }
    },
    limit: 2
  },
  scope2: {
    where: {
      age: {
        [Op.gt]: 30
      }
    },
    limit: 10
  }
}
```

调用  `.scope('scope1', 'scope2')` 将产生以下查询

```sql
WHERE firstName = 'bob' AND age > 30 LIMIT 10
```

注意 `scope2` 将覆盖 `limit` 和 `age`,而 `firstName` 被保留. `limit`,`offset`,`order`,`paranoid`,`lock`和`raw`字段被覆盖,而`where`被浅层合并(意味着相同的键将被覆盖). `include` 的合并策略将在后面讨论.

请注意,多个应用作用域的 `attributes` 键以这样的方式合并,即始终保留 `attributes.exclude`. 这允许合并多个作用域,并且永远不会泄漏最终作用域内的敏感字段.

将查找对象直接传递给作用域模型上的`findAll`(和类似的查找程序)时,适用相同的合并逻辑:

```js
Project.scope('deleted').findAll({
  where: {
    firstName: 'john'
  }
})
```

```sql
WHERE deleted = true AND firstName = 'john'
```

这里的 `deleted` 作用域与 finder 合并. 如果我们要将 `where: { firstName: 'john', deleted: false }`  传递给 finder,那么 `deleted` 作用域将被覆盖.

### 合并 include

Include 是根据包含的模型递归合并的. 这是一个非常强大的合并,在 v5 上添加,并通过示例更好地理解.

考虑四种模型:Foo,Bar,Baz和Qux,具有如下多种关联:

```js
class Foo extends Model {}
class Bar extends Model {}
class Baz extends Model {}
class Qux extends Model {}
Foo.init({ name: Sequelize.STRING }, { sequelize });
Bar.init({ name: Sequelize.STRING }, { sequelize });
Baz.init({ name: Sequelize.STRING }, { sequelize });
Qux.init({ name: Sequelize.STRING }, { sequelize });
Foo.hasMany(Bar, { foreignKey: 'fooId' });
Bar.hasMany(Baz, { foreignKey: 'barId' });
Baz.hasMany(Qux, { foreignKey: 'bazId' });
```

现在,考虑Foo上定义的以下四个作用域:

```js
{
  includeEverything: {
    include: {
      model: this.Bar,
      include: [{
        model: this.Baz,
        include: this.Qux
      }]
    }
  },
  limitedBars: {
    include: [{
      model: this.Bar,
      limit: 2
    }]
  },
  limitedBazs: {
    include: [{
      model: this.Bar,
      include: [{
        model: this.Baz,
        limit: 2
      }]
    }]
  },
  excludeBazName: {
    include: [{
      model: this.Bar,
      include: [{
        model: this.Baz,
        attributes: {
          exclude: ['name']
        }
      }]
    }]
  }
}
```

这四个作用域可以很容易地深度合并,例如通过调用 `Foo.scope('includeEverything', 'limitedBars', 'limitedBazs', 'excludeBazName').findAll()`,这完全等同于调用以下内容:

```js
Foo.findAll({
  include: {
    model: this.Bar,
    limit: 2,
    include: [{
      model: this.Baz,
      limit: 2,
      attributes: {
        exclude: ['name']
      },
      include: this.Qux
    }]
  }
});
```

观察四个作用域如何合并为一个. 根据所包含的模型合并作用域的include. 如果一个作用域包括模型A而另一个作用域包括模型B,则合并结果将包括模型A和B.另一方面,如果两个作用域包括相同的模型A,但具有不同的参数(例如嵌套include或其他属性) ,这些将以递归方式合并,如上所示.

无论应用于作用域的顺序如何,上面说明的合并都以完全相同的方式工作. 如果某个参数由两个不同的作用域设置,那么只会该顺序产生差异 - 这不是上述示例的情况,因为每个作用域都做了不同的事情.

这种合并策略的工作方式与传递给`.findAll`,`.findOne`等的参数完全相同.

## 关联

Sequelize 与关联有两个不同但相关的作用域概念. 差异是微妙但重要的:

* **关联作用域**  允许你在获取和设置关联时指定默认属性 - 在实现多态关联时很有用. 当使用`get`,`set`,`add`和`create`相关联的模型函数时,这个作用域仅在两个模型之间的关联上被调用
* **关联模型上的作用域** 允许你在获取关联时应用默认和其他作用域,并允许你在创建关联时传递作用域模型. 这些作用域都适用于模型上的常规查找和通过关联查找.

举个例子,思考模型Post和Comment. Comment与其他几个模型(图像,视频等)相关联,Comment和其他模型之间的关联是多态的,这意味着除了外键 `commentable_id` 之外,注释还存储一个`commentable`列.

可以使用  _association scope_ 来实现多态关联:

```js
this.Post.hasMany(this.Comment, {
  foreignKey: 'commentable_id',
  scope: {
    commentable: 'post'
  }
});
```

当调用 `post.getComments()` 时,这将自动添加 `WHERE commentable = 'post'`. 类似地,当向帖子添加新的注释时,`commentable` 会自动设置为 `'post'`. 关联作用域是为了存活于后台,没有程序员不必担心 - 它不能被禁用. 有关更完整的多态性示例,请参阅 [关联作用域](associations.html#作用域)

那么考虑那个Post的默认作用域只显示活动的帖子:`where: { active: true }`. 该作用域存在于相关联的模型(Post)上,而不是像`commentable` 作用域那样在关联上. 就像在调用`Post.findAll()` 时一样应用默认作用域,当调用 `User.getPosts()` 时,它也会被应用 - 这只会返回该用户的活动帖子.

要禁用默认作用域,将 `scope: null` 传递给 getter: `User.getPosts({ scope: null })`. 同样,如果要应用其他作用域,请像这样:

```js
User.getPosts({ scope: ['scope1', 'scope2']});
```

如果要为关联模型上的作用域创建快捷方式,可以将作用域模型传递给关联. 考虑一个快捷方式来获取用户所有已删除的帖子:

```js
class Post extends Model {}
Post.init(attributes, {
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
    }
  },
  sequelize,
});

User.hasMany(Post); // 常规 getPosts 关联
User.hasMany(Post.scope('deleted'), { as: 'deletedPosts' });

```

```js
User.getPosts(); // WHERE active = true
User.getDeletedPosts(); // WHERE deleted = true
```
