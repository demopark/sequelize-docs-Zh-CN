# Upgrade to v6 - 升级到 V6

Sequelize v6 是 v5 之后的下一个主要版本. 以下列出了一些重大更改以帮助你进行升级.

## 重大变化

### 支持 Node 10 以及更高

Sequelize v6 将支持 Node 10 及更高版本 [#10821](https://github.com/sequelize/sequelize/issues/10821).

### CLS

你现在应该使用 [cls-hooked](https://github.com/Jeff-Lewis/cls-hooked) 软件包来支持 CLS.

```js
const cls = require('cls-hooked');
const namespace = cls.createNamespace('....');
const Sequelize = require('sequelize');

Sequelize.useCLS(namespace);
```

### 数据库引擎支持

我们已经更新了最低支持的数据库引擎版本. 使用较旧的数据库引擎将显示 `SEQUELIZE0006` 弃用警告。 

### Sequelize

- Bluebird 已被移除. 内部所有方法现在都使用 async/await. 公共 API 现在返回原生 promises. 感谢 [Andy Edwards](https://github.com/jedwards1211) 的重构工作.
- `Sequelize.Promise` 不再被提供.
- `sequelize.import` 方法已被删除. CLI 用户应更新到 `sequelize-cli@6`.
- QueryInterface 和 QueryGenerator 的所有实例都已重命名为它们的小写驼峰别名形式，例如:  queryInterface 和 queryGenerator 用作 '模型' 和 '方言' 上的属性名称时，类名称保持不变.

### 模型

#### `options.returning`

参数 `returning: true` 将不再返回模型中未定义的属性. 之前旧的行为可以通过使用 `returning: ['*']` 来实现.

#### `Model.changed()`

现在,此方法使用 [`_.isEqual`](https://lodash.com/docs/4.17.15#isEqual) 进行相等性测试,并且现在可以识别 JSON 对象. 修改 JSON 对象的嵌套值不会将其标记为已更改(因为它仍然是同一对象).

```js
const instance = await MyModel.findOne();

instance.myJsonField.someProperty = 12345; // 更改为 12345
console.log(instance.changed()); // false

await instance.save(); // 这不会保存任何东西

instance.changed("myJsonField", true);
console.log(instance.changed()); // ['myJsonField']

await instance.save(); // 将会保存
```

#### `Model.bulkCreate()`

这个方法现在抛出 `Sequelize.AggregateError` 而不是 `Bluebird.AggregateError`. 现在, 所有错误都显示为 `errors` 标识.

#### `Model.upsert()`

现在所有语言都支持原生 upsert.

```js
const [instance, created] = await MyModel.upsert({});
```

此方法的签名已更改为 `Promise<Model,boolean | null>`. 第一个索引包含被 upsert 的 `instance`, 第二个索引包含一个布尔值(或`null`), 指示记录是创建还是更新。 对于SQLite/Postgres，`created` 值将始终为 `null`。

- MySQL - 使用 ON DUPLICATE KEY UPDATE 实现
- PostgreSQL - 使用 ON CONFLICT DO UPDATE 实现
- SQLite - 使用 ON CONFLICT DO UPDATE 实现
- MSSQL - 使用 MERGE 语句实现

_<ins>Postgres 用户需要注意:</ins>_ 如果 upsert 有效负载包含 PK 字段, 则 PK 将用作冲突目标. 否则, 将选择第一个唯一约束作为冲突键.

### 查询接口

#### `addConstraint`

现在, 此方法仅使用2个参数, 即 `tableName` 和 `options`. 以前, 第二个参数是要应用约束的列名列表，此列表现在必须作为 `options.fields` 属性传递.