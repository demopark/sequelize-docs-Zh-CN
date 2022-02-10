# Migrations - 迁移

就像你使用[版本控制](https://en.wikipedia.org/wiki/Version_control) 如 [Git](https://en.wikipedia.org/wiki/Git) 来管理源代码的更改一样,你可以使用迁移来跟踪数据库的更改. 通过迁移,你可以将现有的数据库转移到另一个状态,反之亦然:这些状态转换将保存在迁移文件中,它们描述了如何进入新状态以及如何还原更改以恢复旧状态.

你将需要 [Sequelize CLI](https://github.com/sequelize/cli). CLI支持迁移和项目引导.

Sequelize 中的 Migration 是一个 javascript 文件,它导出两个函数 `up` 和 `down`,这些函数指示如何执行迁移和撤消它. 你可以手动定义这些功能,但不必手动调用它们; 它们将由 CLI 自动调用. 在这些函数中,你应该借助 `sequelize.query` 以及 Sequelize 提供给你的其他任何方法,简单地执行所需的任何查询. 除此之外,没有其他神奇的事情.

## 安装 CLI

要安装 Sequelize CLI,请执行以下操作：

```sh
# using npm
npm install --save-dev sequelize-cli
# using yarn
yarn add sequelize-cli --dev
```

有关详细信息,请参见 [CLI GitHub 库](https://github.com/sequelize/cli).

## 项目启动

要创建一个空项目,你需要执行 `init` 命令

```sh
# using npm
npx sequelize-cli init
# using yarn
yarn sequelize-cli init
```

这将创建以下文件夹

- `config`, 包含配置文件,它告诉CLI如何连接数据库
- `models`,包含你的项目的所有模型
- `migrations`, 包含所有迁移文件
- `seeders`, 包含所有种子文件

### 结构

在继续进行之前,我们需要告诉 CLI 如何连接到数据库. 为此,可以打开默认配置文件 `config/config.json`. 看起来像这样:

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
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

请注意,默认情况下,Sequelize CLI 假定使用 mysql. 如果你使用其他方言,则需要更改 `"dialect"` 参数的内容.

现在编辑此文件并设置正确的数据库凭据和方言.对象的键(例如 "development")用于 `model/index.js` 以匹配 `process.env.NODE_ENV`(当未定义时,默认值是 "development").

Sequelize 将为每个方言使用默认的连接端口(例如,对于postgres,它是端口5432). 如果需要指定其他端口,请使用 `port` 字段(默认情况下它不在 config/config.js 中,但你可以简单地添加它).

**注意:** _如果你的数据库还不存在,你可以调用 `db:create` 命令. 通过正确的访问,它将为你创建该数据库._

## 创建第一个模型(和迁移)

一旦你正确配置了CLI配置文件,你就可以首先创建迁移. 它像执行一个简单的命令一样简单.

我们将使用 `model:generate` 命令. 此命令需要两个选项

- `name`: 模型的名称
- `attributes`: 模型的属性列表

让我们创建一个名叫 `User` 的模型

```sh
# using npm
npx sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
# using yarn
yarn sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
```

这将发生以下事情

- 在 `models` 文件夹中创建了一个 `user` 模型文件;
- 在 `migrations` 文件夹中创建了一个名字像 `XXXXXXXXXXXXXX-create-user.js` 的迁移文件.

**注意:** _Sequelize 将只使用模型文件,它是表描述.另一边,迁移文件是该模型的更改,或更具体的是说 CLI 所使用的表. 处理迁移,如提交或日志,以进行数据库的某些更改. _

## 运行迁移

直到这一步,CLI没有将任何东西插入数据库. 我们刚刚为我们的第一个模型 `User` 创建了必需的模型和迁移文件. 现在要在数据库中实际创建该表,需要运行 `db:migrate` 命令.

```sh
# using npm
npx sequelize-cli db:migrate
# using yarn
yarn sequelize-cli db:migrate
```

此命令将执行这些步骤

- 将在数据库中确保一个名为 `SequelizeMeta` 的表. 此表用于记录在当前数据库上运行的迁移
- 开始寻找尚未运行的任何迁移文件. 这可以通过检查 `SequelizeMeta` 表. 在这个例子中,它将运行我们在最后一步中创建的 `XXXXXXXXXXXXXX-create-user.js` 迁移,.
- 创建一个名为 `Users` 的表,其中包含其迁移文件中指定的所有列.

## 撤消迁移

现在我们的表已创建并保存在数据库中. 通过迁移,只需运行命令即可恢复为旧状态.

你可以使用 `db:migrate:undo`,这个命令将会恢复最近的迁移.

```sh
# using npm
npx sequelize-cli db:migrate:undo
# using yarn
yarn sequelize-cli db:migrate:undo
```

通过使用  `db:migrate:undo:all` 命令撤消所有迁移,可以恢复到初始状态. 你还可以通过将其名称传递到 `--to` 选项中来恢复到特定的迁移.

```sh
# using npm
npx sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
# using yarn
yarn sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
```

### 创建第一个种子

假设我们希望在默认情况下将一些数据插入到几个表中. 如果我们跟进前面的例子,我们可以考虑为 `User` 表创建演示用户.

要管理所有数据迁移,你可以使用 `seeders`. 种子文件是数据的一些变化,可用于使用样本数据或测试数据填充数据库表.

让我们创建一个种子文件,它会将一个演示用户添加到我们的 `User` 表中.

```sh
# using npm
npx sequelize-cli seed:generate --name demo-user
# using yarn
yarn sequelize-cli seed:generate --name demo-user
```

这个命令将会在 `seeders` 文件夹中创建一个种子文件.文件名看起来像是  `XXXXXXXXXXXXXX-demo-user.js`,它遵循相同的 `up/down` 语义,如迁移文件.

现在我们应该编辑这个文件,将演示用户插入`User`表.

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert('Users', [{
      firstName: 'John',
      lastName: 'Doe',
      email: 'example@example.com',
      createdAt: new Date(),
      updatedAt: new Date()
    }]);
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete('Users', null, {});
  }
};
```

## 运行种子

在上一步中,你创建了一个种子文件. 但它还没有保存到数据库. 为此,我们需要运行一个简单的命令.

```sh
# using npm
npx sequelize-cli db:seed:all
# using yarn
yarn sequelize-cli db:seed:all
```

这将执行该种子文件,你将有一个演示用户插入 `User` 表.

**注意:** _与使用 `SequelizeMeta` 表的迁移不同,`Seeder` 执行历史记录不会存储在任何地方. 如果你想更改此行为,请阅读 `存储` 部分_

## 撤销种子

Seeders 如果使用了任何存储那么就可以被撤消. 有两个可用的命令

如果你想撤消最近的种子

```sh
# using npm
npx sequelize-cli db:seed:undo
# using yarn
yarn sequelize-cli db:seed:undo
```

如果你想撤消特定的种子

```sh
# using npm
npx sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
# using yarn
yarn sequelize-cli db:seed:undo --seed name-of-seed-as-in-data
```

如果你想撤消所有的种子

```sh
# using npm
npx sequelize-cli db:seed:undo:all
# using yarn
yarn sequelize-cli db:seed:undo:all
```

## 高级专题

以下框架显示了一个典型的迁移文件.

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    // 转变为新状态的逻辑
  },
  down: (queryInterface, Sequelize) => {
    // 恢复更改的逻辑
  }
}
```

我们可以使用 `migration:generate` 生成该文件. 这将在你的迁移文件夹中创建 `xxx-migration-skeleton.js`.

```sh
# using npm
npx sequelize-cli migration:generate --name migration-skeleton
# using yarn
yarn sequelize-cli migration:generate --name migration-skeleton
```

传递的 `queryInterface` 对象可以用来修改数据库. `Sequelize` 对象存储可用的数据类型,如 `STRING` 或 `INTEGER`. 函数 `up` 或 `down` 应该返回一个 `Promise` . 让我们来看一个例子

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Person', {
      name: Sequelize.DataTypes.STRING,
      isBetaMember: {
        type: Sequelize.DataTypes.BOOLEAN,
        defaultValue: false,
        allowNull: false
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
};
```

以下是一个迁移示例,该迁移使用自动管理的事务来在数据库中执行两次更改,以确保成功执行所有指令或在发生故障时回滚所有指令：

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(t => {
      return Promise.all([
        queryInterface.addColumn('Person', 'petName', {
          type: Sequelize.DataTypes.STRING
        }, { transaction: t }),
        queryInterface.addColumn('Person', 'favoriteColor', {
          type: Sequelize.DataTypes.STRING,
        }, { transaction: t })
      ]);
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(t => {
      return Promise.all([
        queryInterface.removeColumn('Person', 'petName', { transaction: t }),
        queryInterface.removeColumn('Person', 'favoriteColor', { transaction: t })
      ]);
    });
  }
};
```

下一个是具有外键的迁移示例. 你可以使用 references 来指定外键:

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Person', {
      name: Sequelize.DataTypes.STRING,
      isBetaMember: {
        type: Sequelize.DataTypes.BOOLEAN,
        defaultValue: false,
        allowNull: false
      },
      userId: {
        type: Sequelize.DataTypes.INTEGER,
        references: {
          model: {
            tableName: 'users',
            schema: 'schema'
          },
          key: 'id'
        },
        allowNull: false
      },
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
}
```

下一个是使用 async/await 的迁移示例, 其中你通过手动管理的事务在新列上创建唯一索引：

```js
module.exports = {
  async up(queryInterface, Sequelize) {
    const transaction = await queryInterface.sequelize.transaction();
    try {
      await queryInterface.addColumn(
        'Person',
        'petName',
        {
          type: Sequelize.DataTypes.STRING,
        },
        { transaction }
      );
      await queryInterface.addIndex(
        'Person',
        'petName',
        {
          fields: 'petName',
          unique: true,
          transaction,
        }
      );
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  },
  async down(queryInterface, Sequelize) {
    const transaction = await queryInterface.sequelize.transaction();
    try {
      await queryInterface.removeColumn('Person', 'petName', { transaction });
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  }
};
```

下一个示例, 该迁移创建具多个字段组成的唯一索引, 该索引允许一个关系存在多次, 但只有一个满足条件:

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    queryInterface.createTable('Person', {
      name: Sequelize.DataTypes.STRING,
      bool: {
        type: Sequelize.DataTypes.BOOLEAN,
        defaultValue: false
      }
    }).then((queryInterface, Sequelize) => {
      queryInterface.addIndex(
        'Person',
        ['name', 'bool'],
        {
          indicesType: 'UNIQUE',
          where: { bool : 'true' },
        }
      );
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
}
```

### `.sequelizerc` 文件

这是一个特殊的配置文件. 它允许你指定通常作为参数传递给CLI的各种选项:

- `env`: 在其中运行命令的环境
- `config`: 配置文件的路径
- `options-path`: 带有其他参数的 JSON 文件的路径
- `migrations-path`: migrations 文件夹的路径
- `seeders-path`: seeders 文件夹的路径
- `models-path`: models 文件夹的路径
- `url`: 要使用的数据库连接字符串. 替代使用 --config 文件
- `debug`: 可用时显示各种调试信息

在某些情况下,你可以使用它:

- 你想要覆盖到 `migrations`, `models`, `seeders` 或 `config` 文件夹的路径.
- 你想要重命名 `config.json` 成为别的名字比如 `database.json`

还有更多的, 让我们看一下如何使用这个文件进行自定义配置.

首先,让我们在项目的根目录中创建 `.sequelizerc` 文件,其内容如下：

```js
// .sequelizerc

const path = require('path');

module.exports = {
  'config': path.resolve('config', 'database.json'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations')
};
```

通过这个配置你告诉CLI: 

- 使用 `config/database.json` 文件来配置设置;
- 使用 `db/models` 作为模型文件夹;
- 使用 `db/seeders` 作为种子文件夹;
- 使用 `db/migrations` 作为迁移文件夹;

### 动态配置

默认情况下,配置文件是一个名为 `config.json` 的 JSON 文件. 但是有时你需要动态配置,例如访问环境变量或执行其他代码来确定配置.

值得庆幸的是,Sequelize CLI 可以从 `.json` 和 `.js` 文件中读取. 可以使用 `.sequelizerc` 文件来设置. 你只需提供 `.js` 文件的路径作为导出对象的 `config` 参数即可：

```js
const path = require('path');

module.exports = {
  'config': path.resolve('config', 'config.js')
}
```

现在,Sequelize CLI 将加载 `config/config.js` 以获取配置参数.

一个 `config/config.js` 文件的例子

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
      bigNumberStrings: true
    }
  },
  test: {
    username: process.env.CI_DB_USERNAME,
    password: process.env.CI_DB_PASSWORD,
    database: process.env.CI_DB_NAME,
    host: '127.0.0.1',
    port: 3306,
    dialect: 'mysql',
    dialectOptions: {
      bigNumberStrings: true
    }
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
        ca: fs.readFileSync(__dirname + '/mysql-ca-main.crt')
      }
    }
  }
};
```

上面的示例还显示了如何向配置中添加自定义方言参数.

### 使用 Babel

为了在你的迁移和 seeder 中实现更现代的构造,你可以简单地安装 `babel-register` 并在 `.sequelizerc` 开始时 require 它：

```sh
# using npm
npm i --save-dev babel-register
# using yarn
yarn add babel-register --dev
```

```js
// .sequelizerc

require("babel-register");

const path = require('path');

module.exports = {
  'config': path.resolve('config', 'config.json'),
  'models-path': path.resolve('models'),
  'seeders-path': path.resolve('seeders'),
  'migrations-path': path.resolve('migrations')
}
```

当然,结果将取决于你的 babel 配置(例如在 `.babelrc` 文件中). 在[babeljs.io](https://babeljs.io)了解更多信息

### 安全提示

使用环境变量进行配置设置. 这是因为诸如密码之类的机密绝不应该是源代码的一部分(尤其是不要提交给版本控制).

### 存储

你可以使用三种类型的存储：`sequelize`,`json`和`none`.

- `sequelize` : 将迁移和种子存储在 sequelize 数据库的表中
- `json` : 将迁移和种子存储在 json 文件中
- `none` : 不存储任何迁移/种子

#### 迁移存储

默认情况下,CLI 将在数据库中创建一个名为 `SequelizeMeta` 的表,其中包含每个执行的迁移的条目. 要更改此行为,可以将三个参数添加到配置文件中. 使用 `migrationStorage`,你可以选择用于迁移的存储类型. 如果选择 `json`,则可以使用 `migrationStoragePath` 指定文件的路径,否则 CLI 将写入文件 `sequelize-meta.json`. 如果你想使用 `sequelize` 将信息保留在数据库中,但又想使用其他表,则可以使用 `migrationStorageTableName` 来更改表名. 你还可以通过提供 `migrationStorageTableSchema` 属性来为 `SequelizeMeta` 表定义不同的架构.

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",

    // 使用其他存储类型. 默认: sequelize
    "migrationStorage": "json",

    // 使用其他文件名. 默认: sequelize-meta.json
    "migrationStoragePath": "sequelizeMeta.json",

    // 使用其他表格名称. 默认: SequelizeMeta
    "migrationStorageTableName": "sequelize_meta",

    // 对 SequelizeMeta 表使用其他架构
    "migrationStorageTableSchema": "custom_schema"
  }
}
```

**Note:** _不建议将 `none` 存储作为迁移存储. 如果你决定使用它,请注意没有任何迁移进行或未运行的记录的含义._

#### 种子储存

默认情况下,CLI 不会保存任何已执行的种子. 如果选择更改此行为(！),则可以在配置文件中使用 `seederStorage` 来更改存储类型. 如果选择 `json`,则可以使用 `seederStoragePath` 指定文件的路径,否则 CLI 将写入文件 `sequelize-data.json`. 如果要使用 `sequelize` 将信息保留在数据库中,则可以使用 `seederStorageTableName` 指定表名,否则它将默认为 `SequelizeData`.

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",
    // 使用其他存储. 默认: none
    "seederStorage": "json",
    // 使用其他文件名称. 默认: sequelize-data.json
    "seederStoragePath": "sequelizeData.json",
    // 使用其他表格名称. 默认: SequelizeData
    "seederStorageTableName": "sequelize_data"
  }
}
```

### 配置连接字符串

配置连接字符串作为配置文件定义数据库的 `--config` 参数的替代方法,可以使用 `--url` 参数来传递连接字符串. 例如：

```sh
# using npm
npx sequelize-cli db:migrate --url 'mysql://root:password@mysql_host.com/database_name'
# using yarn
yarn sequelize-cli db:migrate --url 'mysql://root:password@mysql_host.com/database_name'
```

### 程序用法

程序用法 Sequelize 有一个名为 [umzug](https://github.com/sequelize/umzug) 的姊妹库,用于以编程方式处理迁移任务的执行和日志记录.