# Validations & Constraints - 验证 & 约束

在本教程中,你将学习如何在 Sequelize 中设置模型的验证和约束.

对于本教程,将假定以下设置：

```js
const { Sequelize, Op, Model, DataTypes } = require("sequelize");
const sequelize = new Sequelize("sqlite::memory:");

const User = sequelize.define("user", {
  username: {
    type: DataTypes.TEXT,
    allowNull: false,
    unique: true
  },
  hashedPassword: {
    type: DataTypes.STRING(64),
    is: /^[0-9a-f]{64}$/i
  }
});

(async () => {
  await sequelize.sync({ force: true });
  // 这是代码
})();
```

## 验证和约束的区别

验证是在纯 JavaScript 中在 Sequelize 级别执行的检查. 如果你提供自定义验证器功能,它们可能会非常复杂,也可能是 Sequelize 提供的内置验证器之一. 如果验证失败,则根本不会将 SQL 查询发送到数据库.

另一方面,约束是在 SQL 级别定义的规则. 约束的最基本示例是唯一约束. 如果约束检查失败,则数据库将引发错误,并且 Sequelize 会将错误转发给 JavaScript(在此示例中,抛出 `SequelizeUniqueConstraintError`). 请注意,在这种情况下,与验证不同,它执行了 SQL 查询.

## 唯一约束

下面的代码示例在 `username` 字段上定义了唯一约束：

```js
/* ... */ {
  username: {
    type: DataTypes.TEXT,
    allowNull: false,
    unique: true
  },
} /* ... */
```

同步此模型后(例如,通过调用`sequelize.sync`),在表中将 `username` 字段创建为 `` `username` TEXT UNIQUE``,如果尝试插入已存在的用户名将抛出 `SequelizeUniqueConstraintError`.

## 允许/禁止 null 值

默认情况下,`null` 是模型每一列的允许值. 可以通过为列设置 `allowNull: false` 参数来禁用它,就像在我们的代码示例的 `username` 字段中所做的一样：

```js
/* ... */ {
  username: {
    type: DataTypes.TEXT,
    allowNull: false,
    unique: true
  },
} /* ... */
```

如果没有 `allowNull: false`, 那么调用 `User.create({})` 将会生效.

### 关于 `allowNull` 实现的说明

按照本教程开头所述,`allowNull` 检查是 Sequelize 中唯一由 *验证* 和 *约束* 混合而成的检查. 这是因为：

* 如果试图将 `null` 设置到不允许为 null 的字段,则将抛出`ValidationError` ,而且 *不会执行任何 SQL 查询*.
* 另外,在 `sequelize.sync` 之后,具有 `allowNull: false` 的列将使用 `NOT NULL` SQL 约束进行定义. 这样,尝试将值设置为 `null` 的直接 SQL 查询也将失败.

## 验证器

使用模型验证器,可以为模型的每个属性指定 格式/内容/继承 验证. 验证会自动在 `create`, `update` 和 `save` 时运行. 你还可以调用 `validate()` 来手动验证实例.

### 按属性验证

你可以定义你的自定义验证器,也可以使用由 [validator.js (10.11.0)](https://github.com/chriso/validator.js) 实现的多个内置验证器,如下所示.

```js
sequelize.define('foo', {
  bar: {
    type: DataTypes.STRING,
    validate: {
      is: /^[a-z]+$/i,          // 匹配这个 RegExp
      is: ["^[a-z]+$",'i'],     // 与上面相同,但是以字符串构造 RegExp
      not: /^[a-z]+$/i,         // 不匹配 RegExp
      not: ["^[a-z]+$",'i'],    // 与上面相同,但是以字符串构造 RegExp
      isEmail: true,            // 检查 email 格式 (foo@bar.com)
      isUrl: true,              // 检查 url 格式 (http://foo.com)
      isIP: true,               // 检查 IPv4 (129.89.23.1) 或 IPv6 格式
      isIPv4: true,             // 检查 IPv4 格式 (129.89.23.1)
      isIPv6: true,             // 检查 IPv6 格式
      isAlpha: true,            // 只允许字母
      isAlphanumeric: true,     // 将仅允许使用字母数字,因此 '_abc' 将失败
      isNumeric: true,          // 只允许数字
      isInt: true,              // 检查有效的整数
      isFloat: true,            // 检查有效的浮点数
      isDecimal: true,          // 检查任何数字
      isLowercase: true,        // 检查小写
      isUppercase: true,        // 检查大写
      notNull: true,            // 不允许为空
      isNull: true,             // 只允许为空
      notEmpty: true,           // 不允许空字符串
      equals: 'specific value', // 仅允许 'specific value'
      contains: 'foo',          // 强制特定子字符串
      notIn: [['foo', 'bar']],  // 检查值不是这些之一
      isIn: [['foo', 'bar']],   // 检查值是其中之一
      notContains: 'bar',       // 不允许特定的子字符串
      len: [2,10],              // 仅允许长度在2到10之间的值
      isUUID: 4,                // 只允许 uuid
      isDate: true,             // 只允许日期字符串
      isAfter: "2011-11-05",    // 仅允许特定日期之后的日期字符串
      isBefore: "2011-11-05",   // 仅允许特定日期之前的日期字符串
      max: 23,                  // 仅允许值 <= 23
      min: 23,                  // 仅允许值 >= 23
      isCreditCard: true,       // 检查有效的信用卡号

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
});
```

请注意,在需要将多个参数传递给内置验证函数的情况下,要传递的参数必须位于数组中. 但是,如果要传递单个数组参数,例如,`isIn` 可接受的字符串数组,则将其解释为多个字符串参数,而不是一个数组参数. 要解决此问题,请传递一个单长度的参数数组,例如上面所示的 `[['foo', 'bar']]` .

要使用自定义错误消息而不是 [validator.js](https://github.com/chriso/validator.js) 提供的错误消息,请使用对象而不是纯值或参数数组,例如验证器 不需要参数就可以给自定义消息

```js
isInt: {
  msg: "必须是价格的整数"
}
```

或者如果还需要传递参数,则添加一个 `args` 属性：

```js
isIn: {
  args: [['en', 'zh']],
  msg: "必须为英文或中文"
}
```

使用自定义验证器功能时,错误消息将是抛出的 `Error` 对象所持有的任何消息.

有关内置验证方法的更多详细信息,请参见[validator.js 项目](https://github.com/chriso/validator.js).

**提示:** 你还可以为日志记录部分定义自定义功能. 只需传递一个函数. 第一个参数是记录的字符串.

### `allowNull` 与其他验证器的交互

如果将模型的特定字段设置为不允许为 null(使用 `allowNull: false`),并且该值已设置为 `null`,则将跳过所有验证器,并抛出 `ValidationError`.

另一方面,如果将其设置为允许 null(使用 `allowNull: true`),并且该值已设置为 `null`,则仅会跳过内置验证器,而自定义验证器仍将运行.

举例来说,这意味着你可以拥有一个字符串字段,该字段用于验证其长度在5到10个字符之间,但也允许使用 `null` (因为当该值为 `null` 时,长度验证器将被自动跳过)：

```js
class User extends Model {}
User.init({
  username: {
    type: DataTypes.STRING,
    allowNull: true,
    validate: {
      len: [5, 10]
    }
  }
}, { sequelize });
```

你也可以使用自定义验证器有条件地允许 `null` 值,因为不会跳过它：

```js
class User extends Model {}
User.init({
  age: Sequelize.INTEGER,
  name: {
    type: DataTypes.STRING,
    allowNull: true,
    validate: {
      customValidator(value) {
        if (value === null && this.age !== 10) {
          throw new Error("除非年龄为10,否则名称不能为 null");
        }
      })
    }
  }
}, { sequelize });
```

你可以通过设置 `notNull` 验证器来自定义 `allowNull` 错误消息：

```js
class User extends Model {}
User.init({
  name: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      notNull: {
        msg: '请输入你的名字'
      }
    }
  }
}, { sequelize });
```

### 模型范围内的验证

还可以定义验证,来在特定于字段的验证器之后检查模型. 例如,使用此方法,可以确保既未设置 `latitude` 和 `longitude`,又未同时设置两者. 如果设置了一个但未设置另一个,则失败.

使用模型对象的上下文调用模型验证器方法,如果它们抛出错误,则认为失败,否则将通过. 这与自定义字段特定的验证器相同.

所收集的任何错误消息都将与字段验证错误一起放入验证结果对象中,其关键字以 `validate` 选项对象中验证方法失败的键命名. 即便在任何时候每种模型验证方法都只有一个错误消息,但它会在数组中显示为单个字符串错误,以最大程度地提高与字段错误的一致性.

一个例子:

```js
class Place extends Model {}
Place.init({
  name: Sequelize.STRING,
  address: Sequelize.STRING,
  latitude: {
    type: DataTypes.INTEGER,
    validate: {
      min: -90,
      max: 90
    }
  },
  longitude: {
    type: DataTypes.INTEGER,
    validate: {
      min: -180,
      max: 180
    }
  },
}, {
  sequelize,
  validate: {
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Either both latitude and longitude, or neither!');
      }
    }
  }
})
```

在这种简单的情况下,如果只给定了纬度或经度,而不是同时给出两者, 则不能验证对象. 如果我们尝试构建一个超出范围的纬度且没有经度的对象,则`somePlace.validate()` 可能会返回：

```js
{
  'latitude': ['Invalid number: latitude'],
  'bothCoordsOrNone': ['Either both latitude and longitude, or neither!']
}
```

也可以使用在单个属性上定义的自定义验证程序(例如 `latitude` 属性,通过检查 `(value === null) !== (this.longitude === null)` )来完成此类验证, 但模型范围内的验证方法更为简洁.
