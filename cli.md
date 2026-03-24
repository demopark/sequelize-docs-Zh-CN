* Sequelize CLI


Sequelize CLI 及其文档尚未准备好用于 Sequelize 7。如果你依赖 CLI，请暂时继续使用 Sequelize 6。

鉴于 CLI 的主要用途是执行迁移，你也可以尝试使用 [umzug](https://github.com/sequelize/umzug) 或其他数据库迁移工具。


就像你使用 [版本控制](https://en.wikipedia.org/wiki/Version_control) 系统（如 [Git](https://en.wikipedia.org/wiki/Git)）来管理源代码的变更一样，你也可以使用**迁移（migrations）**来追踪数据库的变更。通过迁移，你可以将现有数据库转变为另一种状态，反之亦然：这些状态转换会被保存在迁移文件中，描述了如何达到新状态以及如何回滚到旧状态。

你需要 [Sequelize 命令行工具（CLI）](https://github.com/sequelize/cli)。CLI 支持迁移和项目初始化。

Sequelize 中的迁移是一个导出 `up` 和 `down` 两个函数的 JavaScript 文件，分别用于执行和撤销迁移。你需要手动定义这些函数，但不需要手动调用它们；CLI 会自动调用。在这些函数中，你可以使用 `sequelize.query` 及 Sequelize 提供的其他方法执行所需的查询。除此之外没有额外的魔法。


## 安装 CLI

安装 Sequelize CLI：

```bash
# 使用 npm
npm install --save-dev sequelize-cli
# 使用 yarn
yarn add sequelize-cli --dev
```

详情见 [CLI GitHub 仓库](https://github.com/sequelize/cli)。

## 项目初始化

要创建一个空项目，需要执行 `init` 命令：

```bash
# 使用 npm
npx sequelize-cli init
# 使用 yarn
yarn sequelize-cli init
```

这将创建以下文件夹：

- `config`，包含配置文件，告诉 CLI 如何连接数据库
- `models`，包含项目的所有模型
- `migrations`，包含所有迁移文件
- `seeders`，包含所有种子文件

### 配置

在继续之前，我们需要告诉 CLI 如何连接数据库。为此，请打开默认的配置文件 `config/config.json`。内容如下：

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```


注意，Sequelize CLI 默认使用 MySQL。如果你使用其他数据库，需要修改 `"dialect"` 选项。

现在请编辑此文件，设置正确的数据库凭据和方言。对象的键（如 "development"）会在 `model/index.js` 中用于匹配 `process.env.NODE_ENV`（未定义时，默认为 "development"）。

Sequelize 会为每种方言使用默认端口（例如 Postgres 默认 5432）。如需指定其他端口，可添加 `"port"` 字段（默认未包含在 `config/config.js` 中，但可自行添加）。

**注意：** _如果数据库尚未创建，可以直接运行 `db:create` 命令。在有权限的情况下，它会为你创建数据库。_


## 创建第一个模型（及迁移）

配置好 CLI 配置文件后，你就可以创建第一个迁移了。只需执行一个简单命令。

我们将使用 `model:generate` 命令。该命令需要两个参数：

- `name`：模型名称；
- `attributes`：模型属性列表。

让我们创建一个名为 `User` 的模型。

```bash
# 使用 npm
npx sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
# 使用 yarn
yarn sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
```

这将会：

- 在 `models` 文件夹下创建一个 `user` 模型文件；
- 在 `migrations` 文件夹下创建一个类似 `XXXXXXXXXXXXXX-create-user.js` 的迁移文件。

**注意：** _Sequelize 只会使用模型文件，它代表数据表。而迁移文件则是对该模型（更准确地说是表）的更改，由 CLI 使用。把迁移当作数据库变更的提交或日志。_


## 编写迁移文件

下面是一个典型的迁移文件骨架：

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    // 用于变更到新状态的逻辑
  },
  down: (queryInterface, Sequelize) => {
    // 用于回滚更改的逻辑
  },
};
```

我们可以用 `migration:generate` 命令生成此文件。这将在迁移文件夹下创建 `xxx-migration-example.js`。

```bash
# 使用 npm
npx sequelize-cli migration:generate --name migration-example
# 使用 yarn
yarn sequelize-cli migration:generate --name migration-example
```

传入的 `queryInterface` 对象可用于修改数据库。`Sequelize` 对象包含可用的数据类型，如 `STRING` 或 `INTEGER`。`up` 和 `down` 函数应返回一个 `Promise`。来看一个例子：


```js
const { DataTypes } = require('@sequelize/core');

module.exports = {
  up: (queryInterface, Sequelize) => {
    // 创建 Person 表
    return queryInterface.createTable('Person', {
      name: DataTypes.STRING,
      isBetaMember: {
        type: DataTypes.BOOLEAN,
        defaultValue: false,
        allowNull: false,
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    // 删除 Person 表
    return queryInterface.dropTable('Person');
  },
};
```


下面是一个在数据库中执行两个更改的迁移示例，使用自动管理的事务确保所有操作要么全部成功，要么全部回滚：

```js
const { DataTypes } = require('@sequelize/core');

module.exports = {
  up: (queryInterface, Sequelize) => {
    // 在事务中添加两列
    return queryInterface.sequelize.transaction(transaction => {
      return Promise.all([
        queryInterface.addColumn(
          'Person',
          'petName',
          {
            type: DataTypes.STRING,
          },
          { transaction },
        ),
        queryInterface.addColumn(
          'Person',
          'favoriteColor',
          {
            type: DataTypes.STRING,
          },
          { transaction },
        ),
      ]);
    });
  },
  down: (queryInterface, Sequelize) => {
    // 在事务中移除两列
    return queryInterface.sequelize.transaction(transaction => {
      return Promise.all([
        queryInterface.removeColumn('Person', 'petName', { transaction }),
        queryInterface.removeColumn('Person', 'favoriteColor', { transaction }),
      ]);
    });
  },
};
```


下一个例子是带有外键的迁移。你可以用 references 指定外键：

```js
const { DataTypes } = require('@sequelize/core');

module.exports = {
  up: queryInterface => {
    // 创建带有外键的 Person 表
    return queryInterface.createTable('Person', {
      name: DataTypes.STRING,
      isBetaMember: {
        type: DataTypes.BOOLEAN,
        defaultValue: false,
        allowNull: false,
      },
      userId: {
        type: DataTypes.INTEGER,
        references: {
          model: {
            tableName: 'users',
            schema: 'schema',
          },
          key: 'id',
        },
        allowNull: false,
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    // 删除 Person 表
    return queryInterface.dropTable('Person');
  },
};
```


下一个例子是使用 async/await 的迁移，在新列上创建唯一索引，并手动管理事务：

```js
const { DataTypes } = require('@sequelize/core');

module.exports = {
  async up(queryInterface) {
    // 手动管理事务，添加列并创建唯一索引
    const transaction = await queryInterface.sequelize.startUnmanagedTransaction();
    try {
      await queryInterface.addColumn(
        'Person',
        'petName',
        {
          type: DataTypes.STRING,
        },
        { transaction },
      );
      await queryInterface.addIndex('Person', 'petName', {
        fields: 'petName',
        unique: true,
        transaction,
      });
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  },
  async down(queryInterface) {
    // 手动管理事务，移除列
    const transaction = await queryInterface.sequelize.startUnmanagedTransaction();
    try {
      await queryInterface.removeColumn('Person', 'petName', { transaction });
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  },
};
```


下一个例子是在多个字段上创建带条件的唯一索引，允许关系多次存在但只有一个满足条件：

```js
const { DataTypes } = require('@sequelize/core');

module.exports = {
  up: queryInterface => {
    // 创建 Person 表并添加带条件的唯一索引
    queryInterface
      .createTable('Person', {
        name: DataTypes.STRING,
        bool: {
          type: DataTypes.BOOLEAN,
          defaultValue: false,
        },
      })
      .then((queryInterface, Sequelize) => {
        queryInterface.addIndex('Person', ['name', 'bool'], {
          type: 'UNIQUE',
          where: { bool: 'true' },
        });
      });
  },
  down: queryInterface => {
    // 删除 Person 表
    return queryInterface.dropTable('Person');
  },
};
```


## 执行迁移

到目前为止，我们还没有向数据库插入任何内容。我们只是为第一个模型 `User` 创建了所需的模型和迁移文件。现在要真正创建该表，需要运行 `db:migrate` 命令。

```bash
# 使用 npm
npx sequelize-cli db:migrate
# 使用 yarn
yarn sequelize-cli db:migrate
```

该命令会执行以下步骤：

- 确保数据库中有一个名为 `SequelizeMeta` 的表。该表用于记录当前数据库已执行的迁移。
- 查找尚未执行的迁移文件（通过检查 `SequelizeMeta` 表）。在本例中，会执行我们上一步创建的 `XXXXXXXXXXXXXX-create-user.js` 迁移。
- 创建名为 `Users` 的表，包含迁移文件中指定的所有列。


## 撤销迁移

现在我们的表已经在数据库中创建并保存。通过迁移，你可以通过运行命令回到旧状态。

你可以使用 `db:migrate:undo`，该命令会撤销最近一次迁移。

```bash
# 使用 npm
npx sequelize-cli db:migrate:undo
# 使用 yarn
yarn sequelize-cli db:migrate:undo
```

你可以用 `db:migrate:undo:all` 命令撤销所有迁移回到初始状态，也可以通过 `--to` 选项撤销到指定迁移。

```bash
# 使用 npm
npx sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
# 使用 yarn
yarn sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
```


### 创建第一个种子数据（Seed）

假设我们想默认向某些表插入一些数据。延续上面的例子，我们可以为 `User` 表创建一个演示用户。

你可以使用种子（seeders）来管理所有数据迁移。种子文件用于向数据库表填充示例或测试数据。

让我们创建一个向 `User` 表添加演示用户的种子文件。

```bash
# 使用 npm
npx sequelize-cli seed:generate --name demo-user
# 使用 yarn
yarn sequelize-cli seed:generate --name demo-user
```

该命令会在 `seeders` 文件夹下创建一个种子文件，文件名类似 `XXXXXXXXXXXXXX-demo-user.js`。它遵循与迁移文件相同的 `up / down` 语义。

现在我们应该编辑此文件，将演示用户插入 `User` 表。

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    // 插入演示用户
    return queryInterface.bulkInsert('Users', [
      {
        firstName: 'John',
        lastName: 'Doe',
        email: 'example@example.com',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    ]);
  },
  down: (queryInterface, Sequelize) => {
    // 删除演示用户
    return queryInterface.bulkDelete('Users', null, {});
  },
};
```


## 执行种子数据

上一步你创建了种子文件，但还未将其应用到数据库。为此我们运行一个简单命令。

```bash
# 使用 npm
npx sequelize-cli db:seed:all
# 使用 yarn
yarn sequelize-cli db:seed:all
```

这将执行该种子文件，并向 `User` 表插入演示用户。

**注意：** _种子的执行历史不会被记录（与迁移不同，迁移会用 `SequelizeMeta` 表记录）。如需更改此行为，请阅读“存储”部分。_


## 撤销种子数据

如果种子使用了存储，则可以撤销。可用两种命令：

如果要撤销最近一次种子：

```bash
# 使用 npm
npx sequelize-cli db:seed:undo
# 使用 yarn
yarn sequelize-cli db:seed:undo
```

如果要撤销指定种子：

```bash
# 使用 npm
npx sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
# 使用 yarn
yarn sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
```

如果要撤销所有种子：

```bash
# 使用 npm
npx sequelize-cli db:seed:undo:all
# 使用 yarn
yarn sequelize-cli db:seed:undo:all
```


### `.sequelizerc` 文件

这是一个特殊的配置文件。它允许你指定通常作为 CLI 参数传递的以下选项：

- `env`：运行命令的环境
- `config`：配置文件路径
- `options-path`：包含额外选项的 JSON 文件路径
- `migrations-path`：迁移文件夹路径
- `seeders-path`：种子文件夹路径
- `models-path`：模型文件夹路径
- `url`：数据库连接字符串（可替代 --config 文件）
- `debug`：显示调试信息（如可用）

常见使用场景：

- 你想覆盖默认的 `migrations`、`models`、`seeders` 或 `config` 文件夹路径。
- 你想将 `config.json` 重命名为其他名称，如 `database.json`

还有更多用法。下面看看如何用此文件自定义配置。

首先，在项目根目录创建 `.sequelizerc` 文件，内容如下：


```js
// .sequelizerc

const path = require('path');

module.exports = {
  config: path.resolve('config', 'database.json'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations'),
};
```


通过此配置，你告诉 CLI：

- 使用 `config/database.json` 作为配置文件；
- 使用 `db/models` 作为模型文件夹；
- 使用 `db/seeders` 作为种子文件夹；
- 使用 `db/migrations` 作为迁移文件夹。


### 动态配置

配置文件默认是名为 `config.json` 的 JSON 文件。但有时你需要动态配置，例如访问环境变量或执行其他代码来确定配置。

幸运的是，Sequelize CLI 可以读取 `.json` 和 `.js` 文件。这可以通过 `.sequelizerc` 文件设置。只需将 `.js` 文件路径作为导出对象的 `config` 选项：

```js
const path = require('path');

module.exports = {
  config: path.resolve('config', 'config.js'),
};
```

现在 Sequelize CLI 会加载 `config/config.js` 获取配置。

`config/config.js` 文件示例：

```js
const fs = require('fs');

module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    port: 3306,
    dialect: 'mysql',
    dialectOptions: {
      bigNumberStrings: true,
    },
  },
  test: {
    username: process.env.CI_DB_USERNAME,
    password: process.env.CI_DB_PASSWORD,
    database: process.env.CI_DB_NAME,
    host: '127.0.0.1',
    port: 3306,
    dialect: 'mysql',
    dialectOptions: {
      bigNumberStrings: true,
    },
  },
  production: {
    username: process.env.PROD_DB_USERNAME,
    password: process.env.PROD_DB_PASSWORD,
    database: process.env.PROD_DB_NAME,
    host: process.env.PROD_DB_HOSTNAME,
    port: process.env.PROD_DB_PORT,
    dialect: 'mysql',
    dialectOptions: {
      bigNumberStrings: true,
      ssl: {
        ca: fs.readFileSync(__dirname + '/mysql-ca-main.crt'),
      },
    },
  },
};
```

上例还展示了如何为配置添加自定义方言选项。


### 使用 Babel

为了在迁移和种子文件中使用更现代的语法，可以安装 `babel-register` 并在 `.sequelizerc` 开头引入：

```bash
# 使用 npm
npm i --save-dev babel-register
# 使用 yarn
yarn add babel-register --dev
```

```js
// .sequelizerc

require('babel-register');

const path = require('path');

module.exports = {
  config: path.resolve('config', 'config.json'),
  'models-path': path.resolve('models'),
  'seeders-path': path.resolve('seeders'),
  'migrations-path': path.resolve('migrations'),
};
```

当然，效果取决于你的 babel 配置（如 `.babelrc` 文件）。详见 [babeljs.io](https://babeljs.io)。


### 安全提示

请使用环境变量存储配置。因为密码等敏感信息不应出现在源码中（尤其不能提交到版本控制）。


### 存储方式

你可以选择三种存储类型：`sequelize`、`json` 和 `none`。

- `sequelize` ：在 sequelize 数据库的表中存储迁移和种子信息
- `json` ：在 json 文件中存储迁移和种子信息
- `none` ：不存储任何迁移/种子信息

#### 迁移存储

默认情况下，CLI 会在数据库中创建一个名为 `SequelizeMeta` 的表，记录每个已执行的迁移。你可以通过配置文件添加以下选项更改此行为。使用 `migrationStorage` 可以选择迁移的存储类型。若选择 `json`，可用 `migrationStoragePath` 指定文件路径，否则默认写入 `sequelize-meta.json`。若想保存在数据库但用不同表名，可用 `migrationStorageTableName`。还可用 `migrationStorageTableSchema` 指定表的 schema。

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",

    // 使用不同的存储类型，默认：sequelize
    "migrationStorage": "json",

    // 使用不同的文件名，默认：sequelize-meta.json
    "migrationStoragePath": "sequelizeMeta.json",

    // 使用不同的表名，默认：SequelizeMeta
    "migrationStorageTableName": "sequelize_meta",

    // 使用不同的 schema
    "migrationStorageTableSchema": "custom_schema"
  }
}
```

**注意：** _不推荐将 `none` 用作迁移存储。如果使用，需了解没有迁移记录的后果。_

#### 种子存储

默认情况下，CLI 不会保存已执行的种子。如果你想更改此行为，可以在配置文件中用 `seederStorage` 更改存储类型。若选择 `json`，可用 `seederStoragePath` 指定文件路径，否则默认写入 `sequelize-data.json`。若想保存在数据库，可用 `seederStorageTableName` 指定表名，否则默认为 `SequelizeData`。

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",
    // 使用不同的存储方式，默认：none
    "seederStorage": "json",
    // 使用不同的文件名，默认：sequelize-data.json
    "seederStoragePath": "sequelizeData.json",
    // 使用不同的表名，默认：SequelizeData
    "seederStorageTableName": "sequelize_data"
  }
}
```


### 配置连接字符串

除了用配置文件和 `--config` 选项定义数据库外，你还可以用 `--url` 选项传递连接字符串。例如：

```bash
# 使用 npm
npx sequelize-cli db:migrate --url 'mysql://root:password@mysql_host.com/database_name'
# 使用 yarn
yarn sequelize-cli db:migrate --url 'mysql://root:password@mysql_host.com/database_name'
```

如果在 `package.json` 脚本中用 npm，使用带参数的命令时要加一个额外的 `--`。
例如：

```json
// package.json

...
  "scripts": {
    "migrate:up": "npx sequelize-cli db:migrate",
    "migrate:undo": "npx sequelize-cli db:migrate:undo"
  },
...
```

命令用法：`npm run migrage:up -- --url <url>`


### 编程方式使用

Sequelize 有一个姊妹库 [umzug](https://github.com/sequelize/umzug)，可用于以编程方式执行和记录迁移任务。
