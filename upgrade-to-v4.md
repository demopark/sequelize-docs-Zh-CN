# Upgrade to V4 - 升级到 V4

Sequelize V4 是一个重要版本，它引入了新的功能和突破性的变化。 大量的 sequelize 代码库已用 ES2015 功能重构。 以下指南列出了从 v3 升级到 v4 的一些更改。查看 [修改日志](https://github.com/sequelize/sequelize/blob/b49f936e9aa316cf4a13bade76585acf4d5d8b04/changelog.md) 查看全部详细列表。

### 突破性变化

- Node 版本: 要使用新的 ES2015 功能，我们现在至少需要 Node4。从现在开始，我们将支持所有当前的LTS版本的Node。
- 计数器缓存插件以及因此关联的计数器缓存选项已被删除。 使用 `afterCreate` 和 `afterDelete` 钩子可以实现相同的行为。
- 删除了MariaDB方言。 这只是围绕 MySQL 的一个浅层包装，所以使用 `dialect:'mysql` 而不是进一步的改变。
- 删除默认的 `REPEATABLE_READ` 事务隔离。 隔离级别现在默认为数据库的级别。 在启动事务时明确地传递所需的隔离级别。
- 删除了对 `pool: false` 的支持。要使用单个连接，请将 `pool.max` 设置为1。
- 删除了对旧连接池配置关键字的支持。 

 以前:

  ```js
  pool: {
    maxIdleTime: 30000,
    minConnections: 20,
    maxConnections: 30
  }
  ```

  现在:

  ```js
  pool: {
    idle: 30000,
    min: 20,
    max: 30
  }
  ```
- （MySQL）当数字太大时，BIGINT 现在被转换为字符串。
- (MySQL) `DECIMAL` 和 `NEWDECIMAL` 类型现在以 String 形式返回，除非

  ```js
  dialectOptions: {
    decimalNumbers: true
  }
  ```
  被指定.
- 删除了对referencesKey的支持，使用了一个引用对象。

  ```js
  references: {
      key: '',
      model: ''
  }
  ```
  
- `classMethods` 和 `instanceMethods` 已被移除。

  以前:
  
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

  现在:

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
 
- `Model.Instance` 和 `instance.Model` 已被移除。要从一个实例访问模型，只需使用 [`instance.constructor`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor)。 示例类 (`Model.Instance`) 现在是模型本身。
- Sequelize 现在使用一个 bluebird 库的独立副本.

    - sequelize返回的 promise 现在是 `Sequelize.Promise` 而不是 bluebird 的全局 `Promise` 实例。
    - CLS 补丁不会影响 bluebird 的全局 promise。当与 `Promise.all` 和其他 bluebird 方法一起使用时，事务不会自动传递给方法。明确地修补 bluebird 实例，可以让 CLS 能够使用 bluebird 方法。

      ```bash
      $ npm install --save cls-bluebird
      ```

      ```js
      const Promise = require('bluebird');
      const Sequelize = require('sequelize');
      const cls = require('continuation-local-storage');
      const ns = cls.createNamespace('transaction-namespace');
      const clsBluebird = require('cls-bluebird');
      clsBluebird(ns, Promise);
      Sequelize.useCLS(ns);
      ```
- `Sequelize.Validator` 现在是 `validator` 库的独立副本
- `DataTypes.DECIMAL` 对于 MySQL 和 Postgres 返回的是字符串.
- `DataTypes.DATE` 现在使用 `DATETIMEOFFSET` 而不是 `DATETIME2` sql数据类型，以防MSSQL记录时区。要将现有的 `DATETIME2` 列迁移到 `DATETIMEOFFSET` 中, 查看 [#7201](https://github.com/sequelize/sequelize/pull/7201#issuecomment-278899803).
- `options.order` 现在只接受数组类型或 Sequelize 方法的值。 原限支持的字符串值（即`{order:'name DESC'}`）已被弃用。
- 使用 `BelongsToMany` 关系 `add / set / create` 设置器现在通过将它们传递为 `options.through` 来设置属性（以前的第二个参数被用作通过属性，现在它被认为是 `through` 作为子选项的选项）。

  以前:
  
  ```js
  user.addProject(project, { status: 'started' })
  ```

  现在:
  
  ```js
  user.addProject(project, { through: { status: 'started' }})
  ```

- `DATEONLY` 现在以 `YYYY-MM-DD` 格式而不是 `Date` 类型返回字符串
- `Model.validate` 实例方法默认运行验证钩子。以前你需要传递 `{ hooks: true }`. 您可以通过传递  `{ hooks: false }` 来覆盖此行为。
- 当验证失败时，来自 `Model.validate` 实例方法的结果将被拒绝。 验证成功后才能实现。
- 原始参数 where, order 和 group 比如 `where: { $raw: '..', order: [{ raw: '..' }], group: [{ raw: '..' }] }` 删除以防止SQL注入攻击。
- `Sequelize.Utils` 不再是公共API的一部分，使用它自己承担风险。
- `Hooks` 现在应返回 promise。 不支持回调。
- `include` 总是一个数组

  之前:
  ```js
  User.findAll({
    include: {
      model: Comment,
      as: 'comments'
    }
  })
  ```
  
  现在:
  ```js
  User.findAll({
    include: [{
      model: Comment,
      as: 'comments'
    }]
  })
  ```

- `where` 在 `include` 中不会使这个 `include` 及其所有父节点都被 `required`。你可以使用下面的 `beforeFind` 全局 Hook 来保持以前的行为：

  ```js
  function whereRequiredLikeInV3(modelDescriptor) {
    if (!modelDescriptor.include) {
      return false;
    }

    return modelDescriptor.include.some(relatedModelDescriptor => {
      const childDescriptorRequired = whereRequiredLikeInV3(
        relatedModelDescriptor,
      );

      if (
        (relatedModelDescriptor.where || childDescriptorRequired) &&
        typeof relatedModelDescriptor.required === 'undefined'
      ) {
        relatedModelDescriptor.required = true;
      }

      return relatedModelDescriptor.required;
    });
  }
  
  const sequelize = new Sequelize(..., {
    ...,
    define: {
      hooks: {
        beforeFind: whereRequiredLikeInV3,
      },
    },
  });
  ```

### 新功能
- `sequelize.sync({ alter: true })` 的初始版本已添加，并使用 `ALTER TABLE` 命令来同步表。 [迁移](http://docs.sequelizejs.com/manual/tutorial/migrations.html) 仍然是首选，应在生产中使用。
- 现在支持添加和删除数据库约束。 现有的 primary，foreignKey 和其他约束现在可以使用迁移来添加/删除 - [查看更多](http://docs.sequelizejs.com/manual/tutorial/migrations.html#addconstraint-tablename-attributes-options-).
- 实例（数据库行）现在是模型的实例，而不是单独类的实例。这意味着你可以替换`User.build()` 用 `new User()` 和 `sequelize.define(attributes, options)` 用

  ```js
  class User extends Sequelize.Model {}
  User.init(attributes, options)
  ```
  
  然后，您可以直接在类中定义自定义方法，类方法和 getter / setter。
  这也使得有更多的使用模式，例如用 [装饰器](https://www.npmjs.com/package/sequelize-decorators).
- 增加了 `DEBUG` 支持。 现在可以使用 `DEBUG = sequelize * node app.js` 为所有 sequelize 操作启用日志记录。 要过滤记录的查询，请使用 `DEBUG=sequelize:sql:mssql sequelize:connection*` 来记录生成的SQL查询，连接信息等。
- `SQLite` 添加了 `JSON` 数据类型支持。
- `UPSERT` 现在使用 `MERGE` 语句支持 `MSSQL`。
- 事务现在完全支持 `MSSQL`。
- `MSSQL` 方言现在支持过滤的索引。

  ```js
  queryInterface.addIndex(
    'Person',
    ['firstname', 'lastname'],
    {
      where: {
        lastname: {
          $ne: null
        }
      }
    }
  )
  ```
