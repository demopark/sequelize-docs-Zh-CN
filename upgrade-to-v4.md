# Upgrade to V4 - 升级到 V4

Sequelize v4 是当前版本，它引入了一些突破性的变化。 大量的 sequelize 代码库已用 ES2015 功能重构。 以下指南列出了从 v3 升级到 v4 的一些更改。

## 更新记录

v4 的完整 [更新记录](https://github.com/sequelize/sequelize/blob/b49f936e9aa316cf4a13bade76585acf4d5d8b04/changelog.md).

## 突破性变化

### Node

要使用新的 ES2015 功能，Sequelize v4 至少需要 Node v4 或更高版本。

### 概括

* Counter Cache 插件以及因此关联的 `counterCache` 选项已被删除。
* 现在删除了 MariaDB 方言。 这只是 MySQL 的一个简单封装。你可以设置``dialect: 'mysql'`` Sequelize 应该能够与 MariaDB 服务器一起工作。
* `Model.Instance` 和 `instance.Model` 被删除。 要从实例访问模型，只需使用 [`instance.constructor`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor). 实例类 (`Model.Instance`) 现在是模型本身。
* Sequelize现在使用 bluebird 库的独立副本。
* 由 sequelize 返回的 Promises 现在是 `Sequelize.Promise` 的实例，而不是全局 bluebird  `Promise` 的实例。
* 池库更新到 `v3`，现在您需要调用 `sequelize.close()` 关闭池。

### 配置 / 参数

* 删除了对旧连接池配置关键字的支持。

  **以前**
  
  ```js
    pool: {
      maxIdleTime: 30000,
      minConnections: 20,
      maxConnections: 30
    }
  ```

  **现在**
  
  ```js
    pool: {
      idle: 30000,
      min: 20,
      max: 30
    }
  ```
* 删除了对 `pool:false` 的支持。 要使用单个连接，请将 `pool.max` 设置为 1。
* 移除对 ``referencesKey`` 的支持，使用引用对象

  ```js
    references: {
      key: '',
      model: ''
    }
  ```

* 从`sequelize.define` 中删除了 `classMethods` 和 `instanceMethods` 选项。 Sequelize 模型现在是ES6类。 你可以像这样设置类/实例级别的方法

  **以前**

  ```js
  const Model = sequelize.define('Model', {
      ...
  }, {
      classMethods: {
          associate: function (model) {...}
      },
      instanceMethods: {
          someMethod: function () { ...}
      }
  });
  ```

  **现在**

  ```js
  const Model = sequelize.define('Model', {
      ...
  });

  // 类方法
  Model.associate = function (models) {
      ...associate the models
  };

  // 实例方法
  Model.prototype.someMethod = function () {..}
  ```

* `options.order` 现在只接受数组类型或 Sequelize 方法的值。 支持的字符串值（即`{order：'name DESC'}`）已被弃用。
* 通过 `BelongsToMany` 关系，`add / set / create` 设置器现在通过将属性设置为`options.through`来设置属性。 (先前的第二个参数被用作 through 属性，现在认为 `through` 是一个子选项的参数)
* Raw 参数 where, order 和 group 如 `where: { $raw: '..', order: [{ raw: '..' }], group: [{ raw: '..' }] }` 已被删除，以防止SQL注入攻击

  **以前**

  ```js
  user.addProject(project, { status: 'started' });
  ```

  **现在**

  ```js
  user.addProject(project, { through: { status: 'started' } });
  ```

### 数据类型

* (MySQL/Postgres) `BIGINT` 现在作为字符串返回
* (MySQL/Postgres) `DECIMAL` 和 `NEWDECIMAL` 类型现在作为字符串返回
* (MSSQL) 在 MSSQL 的情况下，`DataTypes.DATE` 现在使用 `DATETIMEOFFSET` 而不是 `DATETIME2`  sql 数据类型来记录时区. 将现有的 `DATETIME2` 列迁移到 `DATETIMEOFFSET`, 查看 [#7201](https://github.com/sequelize/sequelize/pull/7201#issuecomment-278899803).
* `DATEONLY` 现在以 `YYYY-MM-DD` 格式而不是 `Date` 类型返回字符串

### 事务 / CLS

* 删除了 `autocommit: true` 默认值，显式设置这个选项来让事务自动提交。
* 删除了默认的 `REPEATABLE_READ` 事务隔离。 现在隔离级别默认为数据库的隔离级别。 在启动事务时显式传递所需的隔离级别。
* CLS补丁不会影响全局 bluebird 的 promise。 使用 `Promise.all` 和其他 bluebird 方法时，事务不会自动传递给方法。 明确地修补 bluebird 实例以使 CLS 与 bluebird 方法一起工作。

    ```bash
    $ npm install --save cls-bluebird
    ```

    ```js
    const Sequelize = require('sequelize');
    const Promise = require('bluebird');
    const clsBluebird = require('cls-bluebird');
    const cls = require('continuation-local-storage');

    const ns = cls.createNamespace('transaction-namespace');
    clsBluebird(ns, Promise);

    Sequelize.useCLS(ns);
    ```

### 原始查询

* Sequelize 现在支持所有方言的绑定参数。 如果方言不支持绑定，在v3 `bind` 选项将回退到 `replacements`。 这可能是MySQL / MSSQL的重大改变，现在查询实际上将使用绑定参数而不是替换回退。

### 其它

* `Sequelize.Validator` 现在是 `validator` 库的独立副本。
* `Model.validate` 实例方法现在默认运行验证 hook。 以前你需要传递 `{hooks：true}`。 你可以通过传递 `{hooks：false}` 来覆盖这个行为。
* 当验证失败时，由 `Model.validate` 实例方法产生的 promise 将被拒绝。 它将在验证成功时完成。
* `Sequelize.Utils `不再是公共 API 的一部分，使用它需要您自担风险。
* `Hooks` 现在应该返回 Promises。 回调已被弃用。
* Getters 不会运行 `instance.get({raw：true})`，而使用`instance.get({plain：true})`
* include 中的 `required` 不会传播到 include 链中。

  要获得 v3 兼容的效果，您需要在包含include上设置 `required`。

  **以前**

  ```js
  user.findOne({
    include: {
      model: project,
      include: {
        model: task,
        required: true
      }
    }
  });
  ```

  **现在**

  ```js
  User.findOne({
    include: {
      model: Project,
      required: true,
      include: {
        model: Task,
        required: true
      }
    }
  });

  User.findOne({
    include: {
      model: Project,
      required: true,
      include: {
        model: Task,
        where: { type: 'important' } // 其中 required 需要默认为 true
      }
    }
  });
  ```

或者，您可以添加 `beforeFind` hook 以获得 v3 兼容的行为 -

  ```js
  function propagateRequired(modelDescriptor) {
    let include = modelDescriptor.include;

    if (!include) return false;
    if (!Array.isArray(include)) include = [include];

    return include.reduce((isRequired, descriptor) => {
      const hasRequiredChild = propogateRequired(descriptor);
      if ((descriptor.where || hasRequiredChild) && descriptor.required === undefined) {
        descriptor.required = true;
      }
      return descriptor.required || isRequired;
    }, false);
  }

  const sequelize = new Sequelize(..., {
    ...,
    define: {
      hooks: {
        beforeFind: propagateRequired
      }
    }
  });
  ```