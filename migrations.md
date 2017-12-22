# Migrations - 迁移

就像您使用Git / SVN来管理源代码的更改一样，您可以使用迁移来跟踪数据库的更改。 通过迁移，您可以将现有的数据库转移到另一个状态，反之亦然：这些状态转换将保存在迁移文件中，它们描述了如何进入新状态以及如何还原更改以恢复旧状态。

您将需要[Sequelize CLI][0]。 CLI支持迁移和项目引导。

## 命令行界面

### 安装命令行界面

让我们从安装CLI开始，你可以在 [这里][0] 找到说明。 最推荐的方式是这样安装

```bash
$ npm install --save sequelize-cli
```

### 引导

要创建一个空项目，你需要执行 `init` 命令

```bash
$ node_modules/.bin/sequelize init
```

这将创建以下文件夹

- `config`, 包含配置文件，它告诉CLI如何连接数据库
- `models`,包含您的项目的所有模型
- `migrations`, 包含所有迁移文件
- `seeders`, 包含所有种子文件

#### 结构

在继续进行之前，我们需要告诉 CLI 如何连接到数据库。 为此，可以打开默认配置文件 `config/config.json`。 看起来像这样

```json
{
  development: {
    username: 'root',
    password: null,
    database: 'database_development',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: 'root',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.PROD_DB_USERNAME,
    password: process.env.PROD_DB_PASSWORD,
    database: process.env.PROD_DB_NAME,
    host: process.env.PROD_DB_HOSTNAME,
    dialect: 'mysql'
  }
}
```

现在编辑此文件并设置正确的数据库凭据和方言。

**注意:** _如果你的数据库还不存在，你可以调用 `db:create` 命令。 通过正确的访问，它将为您创建该数据库。_

### 创建第一个模型（和迁移）

一旦您正确配置了CLI配置文件，您就可以首先创建迁移。 它像执行一个简单的命令一样简单。

我们将使用 `model:generate` 命令。 此命令需要两个选项

- `name`, 模型的名称
- `attributes`, 模型的属性列表

让我们创建一个名叫 `User` 的模型

```bash
$ node_modules/.bin/sequelize model:generate --name User --attributes firstName:string,lastName:string,email:string
```

这将发生以下事情

- 在 `models` 文件夹中创建了一个 `user` 模型文件
- 在 `migrations` 文件夹中创建了一个名字像 `XXXXXXXXXXXXXX-create-user.js` 的迁移文件

**注意:** _Sequelize 将只使用模型文件，它是表描述。另一边，迁移文件是该模型的更改，或更具体的是说 CLI 所使用的表。 处理迁移，如提交或日志，以进行数据库的某些更改。 _

### 运行迁移

直到这一步，CLI没有将任何东西插入数据库。 我们刚刚为我们的第一个模型 `User` 创建了必需的模型和迁移文件。 现在要在数据库中实际创建该表，需要运行 `db:migrate` 命令。

```bash
$ node_modules/.bin/sequelize db:migrate
```

此命令将执行这些步骤

- 将在数据库中确保一个名为 `SequelizeMeta` 的表。 此表用于记录在当前数据库上运行的迁移
- 开始寻找尚未运行的任何迁移文件。 这可以通过检查 `SequelizeMeta` 表。 在这个例子中，它将运行我们在最后一步中创建的 `XXXXXXXXXXXXXX-create-user.js` 迁移，。
- 创建一个名为 `Users` 的表，其中包含其迁移文件中指定的所有列。

### 撤消迁移

现在我们的表已创建并保存在数据库中。 通过迁移，只需运行命令即可恢复为旧状态。

您可以使用 `db:migrate:undo`，这个命令将会恢复最近的迁移。

```bash
$ node_modules/.bin/sequelize db:migrate:undo
```

通过使用  `db:migrate:undo:all` 命令撤消所有迁移，可以恢复到初始状态。 您还可以通过将其名称传递到 `--to` 选项中来恢复到特定的迁移。

```bash
$ node_modules/.bin/sequelize db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
```

### 创建第一个种子

假设我们希望在默认情况下将一些数据插入到几个表中。 如果我们跟进前面的例子，我们可以考虑为 `User` 表创建演示用户。

要管理所有数据迁移，您可以使用 `seeders`。 种子文件是数据的一些变化，可用于使用样本数据或测试数据填充数据库表。

让我们创建一个种子文件，它会将一个演示用户添加到我们的 `User` 表中。

```bash
$ node_modules/.bin/sequelize seed:generate --name demo-user
```

这个命令将会在 `seeders` 文件夹中创建一个种子文件。文件名看起来像是  `XXXXXXXXXXXXXX-demo-user.js`，它遵循相同的 `up/down` 语义，如迁移文件。

现在我们应该编辑这个文件，将演示用户插入`User`表。

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert('Users', [{
        firstName: 'John',
        lastName: 'Doe',
        email: 'demo@demo.com'
      }], {});
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete('Users', null, {});
  }
};

```

### 运行种子

在上一步中，你创建了一个种子文件。 但它还没有保存到数据库。 为此，我们需要运行一个简单的命令。

```bash
$ node_modules/.bin/sequelize db:seed:all
```

这将执行该种子文件，您将有一个演示用户插入 `User` 表。

**注意:** _ `seeders` 执行不会存储在任何使用 `SequelizeMeta` 表的迁移的地方。 如果你想覆盖这个，请阅读 `存储` 部分_

### 撤销种子

Seeders 如果使用了任何存储那么就可以被撤消。 有两个可用的命令

如果你想撤消最近的种子

```bash
node_modules/.bin/sequelize db:seed:undo
```

如果你想撤消所有的种子

```bash
node_modules/.bin/sequelize db:seed:undo:all
```

## 高级专题

### 迁移框架

以下框架显示了一个典型的迁移文件。

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

传递的 `queryInterface` 对象可以用来修改数据库。 `Sequelize` 对象存储可用的数据类型，如 `STRING` 或 `INTEGER`。 函数 `up` 或 `down` 应该返回一个 `Promise` 。 让我们来看一个例子


```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Person', {
        name: Sequelize.STRING,
        isBetaMember: {
          type: Sequelize.BOOLEAN,
          defaultValue: false,
          allowNull: false
        }
      });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Person');
  }
}
```

### `.sequelizerc` 文件

这是一个特殊的配置文件。 它允许您指定通常作为参数传递给CLI的各种选项。 在某些情况下，您可以使用它。

- 你想要覆盖到 `migrations`, `models`, `seeders` 或 `config` 文件夹的路径.
- 你想要重命名 `config.json` 成为别的名字比如 `database.json`

还有更多的, 让我们看一下如何使用这个文件进行自定义配置。

对于初学者，可以在项目的根目录中创建一个空文件。

```bash
$ touch .sequelizerc
```

现在可以使用示例配置。

```js
const path = require('path');

module.exports = {
  'config': path.resolve('config', 'database.json'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations')
}
```

通过这个配置你告诉CLI: 

- 使用 `config/database.json` 文件来配置设置
- 使用 `db/models` 作为模型文件夹
- 使用 `db/seeders` 作为种子文件夹
- 使用 `db/migrations` 作为迁移文件夹

### 动态配置

配置文件是默认的一个名为 `config.json` 的JSON文件。 但有时你想执行一些代码或访问环境变量，这在JSON文件中是不可能的。

Sequelize CLI可以从“JSON”和“JS”文件中读取。 这可以用`.sequelizerc`文件设置。 让我们来看一下

首先，您需要在项目的根文件夹中创建一个 `.sequelizerc` 文件。 该文件应该覆盖 `JS` 文件的配置路径。 推荐这个

```js
const path = require('path');

module.exports = {
  'config': path.resolve('config', 'config.js')
}
```

现在，Sequelize CLI将加载 `config/config.js` 以获取配置选项。 由于这是一个JS文件，您可以执行任何代码并导出最终的动态配置文件。

一个 `config/config.js` 文件的例子

```js
const fs = require('fs');

module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: 'database_test',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOSTNAME,
    dialect: 'mysql',
    dialectOptions: {
      ssl: {
        ca: fs.readFileSync(__dirname + '/mysql-ca-master.crt')
      }
    }
  }
};
```

### 使用环境变量

使用CLI，您可以直接访问 `config/config.js` 内的环境变量。 您可以使用 `.sequelizerc` 来告诉CLI使用 `config/config.js` 进行配置。 这在上一节中有所解释。

然后你可以使用正确的环境变量来暴露文件。

```js
module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: process.env.CI_DB_USERNAME,
    password: process.env.CI_DB_PASSWORD,
    database: process.env.CI_DB_NAME,
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.PROD_DB_USERNAME,
    password: process.env.PROD_DB_PASSWORD,
    database: process.env.PROD_DB_NAME,
    host: process.env.PROD_DB_HOSTNAME,
    dialect: 'mysql'
  }
```

### 指定方言选项

有时你想指定一个 dialectOption，如果它是一个通用配置，你可以将其添加到 `config/config.json` 中。 有时你想执行一些代码来获取 dialectOptions，你应该为这些情况使用动态配置文件。

```json
{
    "production": {
        "dialect":"mysql",
        "dialectOptions": {
            "bigNumberStrings": true
        }
    }
}
```

### 生产用途

有关在生产环境中使用CLI和迁移设置的一些提示。

1) 使用环境变量进行配置设置。 这是通过动态配置更好地实现的。 样品生产安全配置可能看起来像

```js
const fs = require('fs');

module.exports = {
  development: {
    username: 'database_dev',
    password: 'database_dev',
    database: 'database_dev',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  test: {
    username: 'database_test',
    password: null,
    database: 'database_test',
    host: '127.0.0.1',
    dialect: 'mysql'
  },
  production: {
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOSTNAME,
    dialect: 'mysql',
    dialectOptions: {
      ssl: {
        ca: fs.readFileSync(__dirname + '/mysql-ca-master.crt')
      }
    }
  }
};
```

我们的目标是为各种数据库秘密使用环境变量，而不是意外检查它们来源控制。

### 存储

可以使用三种类型的存储：`sequelize`，`json`和`none`。

- `sequelize` : 将迁移和种子存储在 sequelize 数据库的表中
- `json` : 将迁移和种子存储在json文件上
- `none` : 不存储任何迁移/种子


#### 迁移存储

默认情况下，CLI 将在您的数据库中创建一个名为 `SequelizeMeta` 的表，其中包含每个执行迁移的条目。 要更改此行为，可以在配置文件中添加三个选项。 使用 `migrationStorage` 可以选择要用于迁移的存储类型。 如果选择 `json`，可以使用 `migrationStoragePath` 指定文件的路径，或者 CLI 将写入 `sequelize-meta.json` 文件。 如果要将数据保存在数据库中，请使用 `sequelize`，但是要使用其他表格，可以使用 `migrationStorageTableName`.

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",

    // 使用不同的存储类型. Default: sequelize
    "migrationStorage": "json",

    // 使用不同的文件名. Default: sequelize-meta.json
    "migrationStoragePath": "sequelizeMeta.json",

    // 使用不同的表名. Default: SequelizeMeta
    "migrationStorageTableName": "sequelize_meta"
  }
}
```

**注意:** _不推荐使用 `none` 存储作为迁移存储。 如果您决定使用它，请注意将会没有任何移动记录或没有运行的记录._

#### 种子储存

默认情况下，CLI 不会保存任何被执行的种子。 如果您选择更改此行为(!)，则可以在配置文件中使用 `seederStorage` 来更改存储类型。 如果选择 `json`，可以使用 `seederStoragePath` 指定文件的路径，或者 CLI 将写入文件 `sequelize-data.json`。 如果要将数据保存在数据库中，请使用 `sequelize`，您可以使用 `seederStorageTableName` 指定表名，否则将默认为`SequelizeData`。

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",
    // 使用不同的存储空间. Default: none
    "seederStorage": "json",
    // 使用不同的文件名. Default: sequelize-data.json
    "seederStoragePath": "sequelizeData.json",
    // 使用不同的表名 Default: SequelizeData
    "seederStorageTableName": "sequelize_data"
  }
}
```

### 配置连接字符串

作为 `--config` 选项的替代方法，可以使用定义数据库的配置文件，您可以使用 `--url` 选项传递连接字符串。 例如：

```bash
$ node_modules/.bin/sequelize db:migrate --url 'mysql://root:password@mysql_host.com/database_name'
```

### 通过SSL连接

确保ssl在 `dialectOptions` 和基本配置中指定。

```json
{
    "production": {
        "dialect":"postgres",
        "ssl": true,
        "dialectOptions": {
            "ssl": true
        }
    }
}
```

### 程序化使用

Sequelize 有一个 [姊妹库][1]，用于以编程方式处理迁移任务的执行和记录。

## 查询界面

使用 `queryInterface` 对象描述之前，您可以更改数据库模式。 查看完整的公共方法列表，它支持 [QueryInterface API][2]


[0]: https://github.com/sequelize/cli
[1]: https://github.com/sequelize/umzug
[2]: /class/lib/query-interface.js~QueryInterface.html
