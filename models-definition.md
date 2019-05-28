# Model definition - 模型定义

要定义模型和表之间的映射,请使用 `define` 方法. 每列必须具有数据类型,请参阅 [datatypes][1] 的更多信息.

```js
class Project extends Model {}
Project.init({
  title: Sequelize.STRING,
  description: Sequelize.TEXT
}, { sequelize, modelName: 'project' });

class Task extends Model {}
Task.init({
  title: Sequelize.STRING,
  description: Sequelize.TEXT,
  deadline: Sequelize.DATE
}, { sequelize, modelName: 'task' })
```

除了[datatypes][1],你可以在每列上设置很多参数.

```js
class Foo extends Model {}
Foo.init({
 // 如果未赋值,则自动设置值为 TRUE
 flag: { type: Sequelize.BOOLEAN, allowNull: false, defaultValue: true},

 // 设置默认时间为当前时间
 myDate: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },

 // 将allowNull设置为false会将NOT NULL添加到列中,
 // 这意味着当列为空时执行查询时将从DB抛出错误. 
 // 如果要在查询DB之前检查值不为空,请查看下面的验证部分.
 title: { type: Sequelize.STRING, allowNull: false},

 // 创建具有相同值的两个对象将抛出一个错误. 唯一属性可以是布尔值或字符串.
 // 如果为多个列提供相同的字符串,则它们将形成复合唯一键.
 uniqueOne: { type: Sequelize.STRING,  unique: 'compositeIndex'},
 uniqueTwo: { type: Sequelize.INTEGER, unique: 'compositeIndex'},

 // unique属性用来创建一个唯一约束.
 someUnique: {type: Sequelize.STRING, unique: true},
 
 // 这与在模型选项中创建索引完全相同.
 {someUnique: {type: Sequelize.STRING}},
 {indexes: [{unique: true, fields: ['someUnique']}]},

 // primaryKey用于定义主键.
 identifier: { type: Sequelize.STRING, primaryKey: true},

 // autoIncrement可用于创建自增的整数列
 incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },

 // 你可以通过'field'属性指定自定义列名称:
 fieldWithUnderscores: { type: Sequelize.STRING, field: 'field_with_underscores' },

 // 这可以创建一个外键:
 bar_id: {
   type: Sequelize.INTEGER,

   references: {
     // 这是引用另一个模型
     model: Bar,

     // 这是引用模型的列名称
     key: 'id',

     // 这声明什么时候检查外键约束. 仅限PostgreSQL.
     deferrable: Sequelize.Deferrable.INITIALLY_IMMEDIATE
   }
 },

 // 仅可以为 MySQL,PostgreSQL 和 MSSQL 的列添加注释
 commentMe: {
   type: Sequelize.INTEGER,

   comment: '这是一个包含注释的列名'
 }
}, {
  sequelize,
  modelName: 'foo'
});
```

注释选项也可以在表上使用, 查看 [model configuration][0].


## 时间戳

默认情况下,Sequelize 会将 `createdAt` 和 `updatedAt` 属性添加到模型中,以便你能够知道数据库条目何时进入数据库以及何时被更新.

请注意,如果你使用 Sequelize 迁移,则需要将 `createdAt` 和 `updatedAt` 字段添加到迁移定义中:

```js
module.exports = {
  up(queryInterface, Sequelize) {
    return queryInterface.createTable('my-table', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },

      // 时间戳
      createdAt: Sequelize.DATE,
      updatedAt: Sequelize.DATE,
    })
  },
  down(queryInterface, Sequelize) {
    return queryInterface.dropTable('my-table');
  },
}

```

如果你不想在模型上使用时间戳,只需要一些时间戳记,或者你正在使用现有的数据库,其中列被命名为别的东西,直接跳转到 [configuration][0] 以查看如何执行此操作.

## 可延迟

当你在 PostgreSQL 中指定外键列的参数来声明成一个可延迟类型. 可用的选项如下:

```js
// 将所有外键约束检查推迟到事务结束时.
Sequelize.Deferrable.INITIALLY_DEFERRED

// 立即检查外键约束.
Sequelize.Deferrable.INITIALLY_IMMEDIATE

// 不要推迟检查.
Sequelize.Deferrable.NOT
```

最后一个参数是 PostgreSQL 的默认值,不允许你在事务中动态的更改规则. 查看 [事务](transactions.md#参数) 章节获取补充信息.

## Getters & setters

可以在模型上定义'对象属性'getter和setter函数,这些可以用于映射到数据库字段的“保护”属性,也可以用于定义“伪”属性.

Getters和Setters可以通过两种方式定义(你可以混合使用这两种方式):

* 作为属性定义的一部分
* 作为模型参数的一部分

**注意:** 如果在两个地方定义了getter或setter,那么在相关属性定义中找到的函数始终是优先的.

### 定义为属性定义的一部分

```js
class Employee extends Model {}
Employee.init({
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    get() {
      const title = this.getDataValue('title');
      // 'this' 允许你访问实例的属性
      return this.getDataValue('name') + ' (' + title + ')';
    },
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    set(val) {
      this.setDataValue('title', val.toUpperCase());
    }
  }
}, { sequelize, modelName: 'employee' });

Employee
  .create({ name: 'John Doe', title: 'senior engineer' })
  .then(employee => {
    console.log(employee.get('name')); // John Doe (SENIOR ENGINEER)
    console.log(employee.get('title')); // SENIOR ENGINEER
  })
```

### 定义为模型参数的一部分

以下是在模型参数中定义 getter 和 setter 的示例.

`fullName` getter,是一个说明如何在模型上定义伪属性的例子 - 这些属性实际上不是数据库模式的一部分. 事实上,伪属性可以通过两种方式定义:使用模型getter,或者使用[`虚拟`数据类型](http://docs.sequelizejs.com/variable/index.html#static-variable-DataTypes)的列. 虚拟数据类型可以有验证,而虚拟属性的getter则不能.

请注意,`fullName` getter函数中引用的`this.firstname`和`this.lastname`将触发对相应getter函数的调用. 如果你不想这样,可以使用`getDataValue()`方法来访问原始值(见下文).

```js
class Foo extends Model {
  get fullName() {
    return this.firstname + ' ' + this.lastname;
  }

  set fullName(value) {
    const names = value.split(' ');
    this.setDataValue('firstname', names.slice(0, -1).join(' '));
    this.setDataValue('lastname', names.slice(-1).join(' '));
  }
}
Foo.init({
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  sequelize,
  modelName: 'foo'
});

// 或使用 `sequelize.define`
sequelize.define('Foo', {
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  getterMethods: {
    fullName() {
      return this.firstname + ' ' + this.lastname;
    }
  },

  setterMethods: {
    fullName(value) {
      const names = value.split(' ');

      this.setDataValue('firstname', names.slice(0, -1).join(' '));
      this.setDataValue('lastname', names.slice(-1).join(' '));
    }
  }
});
```

### 用于 getter 和 setter 定义内部的 Helper 方法

* 检索底层属性值 - 总是使用 `this.getDataValue()`

```js
/* 一个用于 'title' 属性的 getter */
get() {
  return this.getDataValue('title')
}
```

* 设置基础属性值 - 总是使用 `this.setDataValue()`

```js
/* 一个用于 'title' 属性的 setter */
set(title) {
  this.setDataValue('title', title.toString().toLowerCase());
}
```


**注意:** 坚持使用 `setDataValue()` 和 `getDataValue()` 函数(而不是直接访问底层的“数据值”属性)是非常重要的 - 这样做可以保护你的定制getter和setter不受底层模型实现的变化.

## 验证

模型验证允许你为模型的每个属性指定格式/内容/继承验证.

验证会自动运行在 `create` , `update` 和 `save` 上. 你也可以调用 `validate()` 手动验证一个实例.

### 属性验证器

你可以自定义验证器或使用由[validator.js][3]实现的几个内置验证器,如下所示.

```js
class ValidateMe extends Model {}
ValidateMe.init({
  bar: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // 只允许字母
      is: /^[a-z]+$/i,          // 与上一个示例相同,使用了真正的正则表达式
      not: ["[a-z]",'i'],       // 不允许字母
      isEmail: true,            // 检查邮件格式 (foo@bar.com)
      isUrl: true,              // 检查连接格式 (http://foo.com)
      isIP: true,               // 检查 IPv4 (129.89.23.1) 或 IPv6 格式
      isIPv4: true,             // 检查 IPv4 (129.89.23.1) 格式
      isIPv6: true,             // 检查 IPv6 格式
      isAlpha: true,            // 只允许字母
      isAlphanumeric: true,     // 只允许使用字母数字
      isNumeric: true,          // 只允许数字
      isInt: true,              // 检查是否为有效整数
      isFloat: true,            // 检查是否为有效浮点数
      isDecimal: true,          // 检查是否为任意数字
      isLowercase: true,        // 检查是否为小写
      isUppercase: true,        // 检查是否为大写
      notNull: true,            // 不允许为空
      isNull: true,             // 只允许为空
      notEmpty: true,           // 不允许空字符串
      equals: 'specific value', // 只允许一个特定值
      contains: 'foo',          // 检查是否包含特定的子字符串
      notIn: [['foo', 'bar']],  // 检查是否值不是其中之一
      isIn: [['foo', 'bar']],   // 检查是否值是其中之一
      notContains: 'bar',       // 不允许包含特定的子字符串
      len: [2,10],              // 只允许长度在2到10之间的值
      isUUID: 4,                // 只允许uuids
      isDate: true,             // 只允许日期字符串
      isAfter: "2011-11-05",    // 只允许在特定日期之后的日期字符串
      isBefore: "2011-11-05",   // 只允许在特定日期之前的日期字符串
      max: 23,                  // 只允许值 <= 23
      min: 23,                  // 只允许值 >= 23
      isCreditCard: true,       // 检查有效的信用卡号码

      // 自定义验证器的示例:
      isEven(value) {
        if (parseInt(value) % 2 !== 0) {
          throw new Error('Only even values are allowed!');
        }
      }
      isGreaterThanOtherField(value) {
        if (parseInt(value) <= parseInt(this.otherField)) {
          throw new Error('Bar must be greater than otherField.');
        }
      }
    }
  }
}, { sequelize });
```

请注意,如果需要将多个参数传递给内置的验证函数,则要传递的参数必须位于数组中. 但是,如果要传递单个数组参数,例如`isIn`的可接受字符串数组,则将被解释为多个字符串参数,而不是一个数组参数. 要解决这个问题,传递一个单一长度的参数数组,比如`[['one','two']]`.

要使用自定义错误消息而不是 [validator.js][3] 提供的错误消息,请使用对象而不是纯值或参数数组,例如不需要参数的验证器可以被给定自定义消息:

```js
isInt: {
  msg: "Must be an integer number of pennies"
}
```

或者如果还需要传递参数,请添加一个 `args` 属性:

```js
isIn: {
  args: [['en', 'zh']],
  msg: "Must be English or Chinese"
}
```

当使用自定义验证器函数时,错误消息将是抛出的 `Error` 对象所持有的任何消息.

有关内置验证方法的更多详细信息,请参阅 [validator.js project][3] .

**提示: **你还可以为日志记录部分定义自定义函数. 只是传递一个方法. 第一个参数将是记录的字符串.

### 属性验证器 与 `allowNull`

如果模型的特定字段设置为不允许null(使用`allowNull:false`)并且该值已设置为 `null`,则将跳过所有验证器并抛出 `ValidationError`.

另一方面,如果将其设置为允许null(使用 `allowNull:true`)并且该值已设置为 `null`,则只会跳过内置验证器,而自定义验证器仍将运行.

例如,这意味着你可以使用一个字符串字段来验证其长度在5到10个字符之间,但也允许 `null`(因为当值为 `null` 时,将自动跳过长度验证器):

```js
class User extends Model {}
User.init({
  username: {
    type: Sequelize.STRING,
    allowNull: true,
    validate: {
      len: [5, 10]
    }
  }
}, { sequelize });
```

你还可以使用自定义验证器有条件地允许 `null` 值,因为它不会被跳过:

```js
class User extends Model {}
User.init({
  age: Sequelize.INTEGER,
  name: {
    type: Sequelize.STRING,
    allowNull: true,
    validate: {
      customValidator(value) {
        if (value === null && this.age !== 10) {
          throw new Error("name can't be null unless age is 10");
        }
      })
    }
  }
}, { sequelize });
```

你可以通过设置 `notNull` 验证器来自定义 `allowNull` 错误消息:

```js
class User extends Model {}
User.init({
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    validate: {
      notNull: {
        msg: 'Please enter your name'
      }
    }
  }
}, { sequelize });
```

### 模型范围验证

验证器也可以在特定字段验证器之后用来定义检查模型.例如,你可以确保`纬度`和`经度`都不设置,或者两者都设置,如果设置了一个而另一个未设置则验证失败.

模型验证器方法与模型对象的上下文一起调用,如果它们抛出错误,则认为失败,否则通过. 这与自定义字段特定的验证器一样.

所收集的任何错误消息都将与验证结果对象一起放在字段验证错误中,这个错误使用在`validate`参数对象中以失败的验证方法的键来命名.即便在任何一个时刻,每个模型验证方法只能有一个错误消息,它会在数组中显示为单个字符串错误,以最大化与字段错误的一致性.

一个例子:

```js
class Pub extends Model {}
Pub.init({
  name: { type: Sequelize.STRING },
  address: { type: Sequelize.STRING },
  latitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -90, max: 90 }
  },
  longitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -180, max: 180 }
  },
}, {
  validate: {
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Require either both latitude and longitude or neither')
      }
    }
  },
  sequelize,
})
```

在这种简单情况下,如果给定纬度或经度,而不是同时包含两者,则验证失败. 如果我们尝试构建一个超范围的纬度和经度,那么`raging_bullock_arms.validate()`可能会返回

```js
{
  'latitude': ['Invalid number: latitude'],
  'bothCoordsOrNone': ['Require either both latitude and longitude or neither']
}
```

这样的验证也可以通过在单个属性上定义的自定义验证器(例如`latitude`属性,通过检查`(value === null) !== (this.longitude === null)`)来完成, 但模型范围的验证方法更清晰.

## 配置

你还可以修改 Sequelize 处理列名称的方式:

```js
class Bar extends Model {}
Bar.init({ /* bla */ }, {
  // 模型的名称. 该模型将以此名称存储在`sequelize.models`中.
  // 在这种情况下,默认为类名,即Bar. 
  // 这将控制自动生成的foreignKey和关联命名的名称
  modelName: 'bar',
  // 不添加时间戳属性 (updatedAt, createdAt)
  timestamps: false,

  // 不删除数据库条目,但将新添加的属性deletedAt设置为当前日期(删除完成时). 
  // paranoid 只有在启用时间戳时才能工作
  paranoid: true,

  // 将自动设置所有属性的字段参数为下划线命名方式.
  // 不会覆盖已经定义的字段选项
  underscored: true,

  // 禁用修改表名; 默认情况下,sequelize将自动将所有传递的模型名称(define的第一个参数)转换为复数. 如果你不想这样,请设置以下内容
  freezeTableName: true,

  // 定义表的名称
  tableName: 'my_very_custom_table_name',

  // 启用乐观锁定. 启用时,sequelize将向模型添加版本计数属性,
  // 并在保存过时的实例时引发OptimisticLockingError错误.
  // 设置为true或具有要用于启用的属性名称的字符串.
    version: true,

  // Sequelize 实例
  sequelize,
})
```

如果你希望sequelize处理时间戳,但只想要其中一部分,或者希望你的时间戳被称为别的东西,则可以单独覆盖每个列:

```js
class Foo extends Model {}
Foo.init({ /* bla */ }, {
  // 不要忘记启用时间戳！
  timestamps: true,

  // 我不想要 createdAt
  createdAt: false,

  // 我想 updateAt 实际上被称为 updateTimestamp
  updatedAt: 'updateTimestamp',

  // 并且希望 deletedA t被称为 destroyTime(请记住启用paranoid以使其工作)
  deletedAt: 'destroyTime',
  paranoid: true,

  sequelize,
})
```

你也可以更改数据库引擎,例如 变更到到MyISAM, 默认值是InnoDB.

```js
class Person extends Model {}
Person.init({ /* attributes */ }, {
  engine: 'MYISAM',
  sequelize
})

// 或全局的
const sequelize = new Sequelize(db, user, pw, {
  define: { engine: 'MYISAM' }
})
```

最后,你可以为MySQL和PG中的表指定注释

```js
class Person extends Model {}
Person.init({ /* attributes */ }, {
  comment: "我是一个表注释!",
  sequelize
})
```

## 导入

你还可以使用`import`方法将模型定义存储在单个文件中. 返回的对象与导入文件的功能中定义的完全相同. 由于Sequelize`v1:5.0`的导入是被缓存的,所以当调用文件导入两次或更多次时,不会遇到问题.

```js
// 在你的服务器文件中 - 例如 app.js
const Project = sequelize.import(__dirname + "/path/to/models/project")

// 模型已经在 /path/to/models/project.js 中定义好
// 你可能会注意到,DataTypes与上述相同
module.exports = (sequelize, DataTypes) => {
  class Project extends sequelize.Model { }
  Project.init({
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  }, { sequelize });
  return Project;
}
```

`import`方法也可以接受回调作为参数.

```js
sequelize.import('project', (sequelize, DataTypes) => {
  class Project extends sequelize.Model {}
  Project.init({
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  }, { sequelize })
  return Project;
})
```

这个额外的功能也是有用的, 例如 `Error: Cannot find module` 被抛出,即使 `/path/to/models/project` 看起来是正确的. 一些框架,如 Meteor,重载 `require`,并给出“惊喜”的结果,如:

```text
Error: Cannot find module '/home/you/meteorApp/.meteor/local/build/programs/server/app/path/to/models/project.js'
```

这通过传入Meteor的`require`版本来解决. 所以,虽然这可能会失败 ...

```js
const AuthorModel = db.import('./path/to/models/project');
```
... 这应该是成功的 ...

```js
const AuthorModel = db.import('project', require('./path/to/models/project'));
```



## 乐观锁定

Sequelize 内置支持通过模型实例版本计数的乐观锁定.

默认情况下禁用乐观锁定,可以通过在特定模型定义或全局模型配置中将`version`属性设置为true来启用. 有关详细信息,请参阅[模型配置][0].

乐观锁定允许并发访问模型记录以进行编辑,并防止冲突覆盖数据. 它通过检查另一个进程是否已经读取记录而进行更改,并在检测到冲突时抛出一个OptimisticLockError.

## 数据库同步

当开始一个新的项目时,你还不会有一个数据库结构,并且使用Sequelize你也不需要它. 只需指定你的模型结构,并让库完成其余操作. 目前支持的是创建和删除表:

```js
// 创建表:
Project.sync()
Task.sync()

// 强制创建!
Project.sync({force: true}) // 这将先丢弃表,然后重新创建它

// 删除表:
Project.drop()
Task.drop()

// 事件处理:
Project.[sync|drop]().then(() => {
  // 好吧...一切都很好！
}).catch(error => {
  // oooh,你输入了错误的数据库凭据？
})
```

因为同步和删除所有的表可能要写很多行,你也可以让Sequelize来为做这些:

```js
// 同步所有尚未在数据库中的模型
sequelize.sync()

// 强制同步所有模型
sequelize.sync({force: true})

// 删除所有表
sequelize.drop()

// 广播处理:
sequelize.[sync|drop]().then(() => {
  // woot woot
}).catch(error => {
  // whooops
})
```

因为`.sync({ force: true })`是具有破坏性的操作,可以使用`match`参数作为附加的安全检查.

`match`参数可以通知Sequelize,以便在同步之前匹配正则表达式与数据库名称 - 在测试中使用`force:true`但不使用实时代码的情况下的安全检查.

```js
// 只有当数据库名称以'_test'结尾时,才会运行.sync()
sequelize.sync({ force: true, match: /_test$/ });
```

## 扩展模型

Sequelize 模型是ES6类. 你可以轻松添加自定义实例或类级别的方法.

```js
class User extends Model {
  // 添加一个类级别的方法
  static classLevelMethod() {
    return 'foo';
  }

  // 添加实例级别方法
  instanceLevelMethod() {
    return 'bar';
  }
}
User.init({ firstname: Sequelize.STRING }, { sequelize });
```

当然,你还可以访问实例的数据并生成虚拟的getter:

```js
class User extends Model {
  getFullname() {
    return [this.firstname, this.lastname].join(' ');
  }
}
User.init({ firstname: Sequelize.STRING, lastname: Sequelize.STRING }, { sequelize });

// 示例:
User.build({ firstname: 'foo', lastname: 'bar' }).getFullname() // 'foo bar'
```

### 索引
Sequelize支持在 `Model.sync()` 或 `sequelize.sync` 中创建的模型定义中添加索引.

```js
class User extends Model {}
User.init({}, {
  indexes: [
    // 在 email 上创建一个唯一索引
    {
      unique: true,
      fields: ['email']
    },

    // 在使用 jsonb_path_ops 的 operator 数据上创建一个 gin 索引
    {
      fields: ['data'],
      using: 'gin',
      operator: 'jsonb_path_ops'
    },

    // 默认的索引名将是 [table]_[fields]
    // 创建多列局部索引
    {
      name: 'public_by_author',
      fields: ['author', 'status'],
      where: {
        status: 'public'
      }
    },

    // 具有有序字段的BTREE索引
    {
      name: 'title_index',
      method: 'BTREE',
      fields: ['author', {attribute: 'title', collate: 'en_US', order: 'DESC', length: 5}]
    }
  ],
  sequelize
});
```

[0]: /manual/models-definition.html#configuration
[1]: /manual/data-types.html
[3]: https://github.com/chriso/validator.js
[5]: /docs/final/misc#asynchronicity