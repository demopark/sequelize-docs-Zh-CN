# Getters, Setters & Virtuals - 获取器, 设置器 & 虚拟字段

Sequelize 允许你为模型的属性定义自定义获取器和设置器.

Sequelize 还允许你指定所谓的 *虚拟属性*,它们是 Sequelize 模型上的属性,这些属性在基础 SQL 表中实际上并不存在,而是由 Sequelize 自动填充. 它们对于创建自定义属性非常有用, 这也可以简化你的代码.

## 获取器

获取器是为模型定义中的一列定义的 `get()` 函数：

```js
const User = sequelize.define('user', {
  // 假设我们想要以大写形式查看每个用户名,
  // 即使它们在数据库本身中不一定是大写的
  username: {
    type: DataTypes.STRING,
    get() {
      const rawValue = this.getDataValue('username');
      return rawValue ? rawValue.toUpperCase() : null;
    }
  }
});
```

就像标准 JavaScript 获取器一样,在读取字段值时会自动调用此获取器：

```js
const user = User.build({ username: 'SuperUser123' });
console.log(user.username); // 'SUPERUSER123'
console.log(user.getDataValue('username')); // 'SuperUser123'
```

注意,尽管上面记录为 `SUPERUSER123`,但是真正存储在数据库中的值仍然是 `SuperUser123`. 我们使用了 `this.getDataValue('username')` 来获得该值,并将其转换为大写.

如果我们尝试在获取器中使用 `this.username`,我们将陷入无限循环！ 这就是为什么 Sequelize 提供 `getDataValue` 方法的原因.

## 设置器

设置器是为模型定义中的一列定义的 `set()` 函数. 它接收要设置的值：

```js
const User = sequelize.define('user', {
  username: DataTypes.STRING,
  password: {
    type: DataTypes.STRING,
    set(value) {
      // 在数据库中以明文形式存储密码是很糟糕的.
      // 使用适当的哈希函数来加密哈希值更好.
      this.setDataValue('password', hash(value));
    }
  }
});
```

```js
const user = User.build({ username: 'someone', password: 'NotSo§tr0ngP4$SW0RD!' });
console.log(user.password); // '7cfc84b8ea898bb72462e78b4643cfccd77e9f05678ec2ce78754147ba947acc'
console.log(user.getDataValue('password')); // '7cfc84b8ea898bb72462e78b4643cfccd77e9f05678ec2ce78754147ba947acc'
```

Sequelize 在将数据发送到数据库之前自动调用了设置器. 数据库得到的唯一数据是已经散列过的值.

如果我们想将模型实例中的另一个字段包含在计算中,那也是可以的,而且非常容易！

```js
const User = sequelize.define('user', {
  username: DataTypes.STRING,
  password: {
    type: DataTypes.STRING,
    set(value) {
      // 在数据库中以明文形式存储密码是很糟糕的.
      // 使用适当的哈希函数来加密哈希值更好.
      // 使用用户名作为盐更好.
      this.setDataValue('password', hash(this.username + value));
    }
  }
});
```

**注意：** 上面涉及密码处理的示例尽管比单纯以明文形式存储密码要好得多,但远非完美的安全性. 正确处理密码很困难,这里的所有内容只是为了举例说明 Sequelize 功能. 我们建议让网络安全专家阅读 [OWASP](https://www.owasp.org/) 文档或者访问 [InfoSec StackExchange](https://security.stackexchange.com/).

## 组合获取器和设置器

获取器和设置器都可以在同一字段中定义.

举个例子,假设我们正在建一个 `Post` 模型,其 `content` 是无限长度的文本. 假设要提高内存使用率,我们要存储内容的压缩版本.

*注意：在这种情况下,现代数据库应会自动进行一些压缩. 这只是为了举例.*

```js
const { gzipSync, gunzipSync } = require('zlib');

const Post = sequelize.define('post', {
  content: {
    type: DataTypes.TEXT,
    get() {
      const storedValue = this.getDataValue('content');
      const gzippedBuffer = Buffer.from(storedValue, 'base64');
      const unzippedBuffer = gunzipSync(gzippedBuffer);
      return unzippedBuffer.toString();
    },
    set(value) {
      const gzippedBuffer = gzipSync(value);
      this.setDataValue('content', gzippedBuffer.toString('base64'));
    }
  }
});
```

通过上述设置,每当我们尝试与 `Post` 模型的 `content` 字段进行交互时,Sequelize 都会自动处理自定义的获取器和设置器. 例如：

```js
const post = await Post.create({ content: 'Hello everyone!' });

console.log(post.content); // 'Hello everyone!'
// 一切都在幕后进行,所以我们甚至都可以忘记内容实际上是
// 作为 gzip 压缩的 base64 字符串存储的！

// 但是,如果我们真的很好奇,我们可以获取 'raw' 数据...
console.log(post.getDataValue('content'));
// Output: 'H4sIAAAAAAAACvNIzcnJV0gtSy2qzM9LVQQAUuk9jQ8AAAA='
```

## 虚拟字段

虚拟字段是 Sequelize 在后台填充的字段,但实际上它们不存在于数据库中.

例如,假设我们有一个 User 的 `firstName` 和 `lastName` 属性.\

*同样,这[仅是为了示例](https://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/).*

如果有一种简单的方法能直接获取 *全名* 那会非常好！ 我们可以将 `getters` 的概念与 Sequelize 针对这种情况提供的特殊数据类型结合使用：`DataTypes.VIRTUAL`:

```js
const { DataTypes } = require('@sequelize/core');

const User = sequelize.define('user', {
  firstName: DataTypes.TEXT,
  lastName: DataTypes.TEXT,
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
      throw new Error('不要尝试设置 `fullName` 的值!');
    }
  }
});
```

`VIRTUAL` 字段不会导致数据表也存在此列. 换句话说,上面的模型虽然没有 `fullName` 列. 但是它似乎存在着！

```js
const user = await User.create({ firstName: 'John', lastName: 'Doe' });
console.log(user.fullName); // 'John Doe'
```
