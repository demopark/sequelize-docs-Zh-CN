# Upgrade to V5 - 升级到 V5

Sequelize v5 是 v4 之后的下一个主要版本

### 突破性变化

### 支持 Node 6 以及更高版本

Sequelize v5 将仅支持 Node 6 以及更高版本 [#9015](https://github.com/sequelize/sequelize/issues/9015)

### 安全的运算符

在 v4 中，你会开始收到弃用警告 `String based operators are now deprecated (基于字符串的运算符现在已弃用)`。 同时也介绍了运算符的概念。这些运算符是一些防止散列注入攻击的符号。

**对于 v5**

- 运算符现在默认启用。
- 你仍然可以通过在 `operatorsAliases` 中传递一个运算符映射来使用字符串运算符，但这会产生弃用警告。
- `Op.$raw` 已被移除。

### 模型

**属性**

`Model.attributes` 现在已被移除, 请使用 `Model.rawAttributes`。 [#5320](https://github.com/sequelize/sequelize/issues/5320)

__注意__: _请不要将它与 `options.attributes` 混淆，它们仍然有效_

**偏执模式**

对于 v5 如果设置了 `deletedAt`， 该数据将被视为已删除. `paranoid` 选项只会使用 `deletedAt` 作为标志。 [#8496](https://github.com/sequelize/sequelize/issues/8496)

**Model.bulkCreate**

`updateOnDuplicate` 选项用于接受布尔值和数组，现在只接受非空的数组属性。 [#9288](https://github.com/sequelize/sequelize/issues/9288)


**下划线模式**

`Model.options.underscored` 的实现方式被改变了。你可以在 [这里](https://github.com/sequelize/sequelize/issues/6423#issuecomment-379472035) 找到完整的说明 。

大致内容

1. `underscoredAll` 和 `underscored` 选项都合并为一个 `underscored` 选项
2. 现在，所有属性都默认使用 camelcase 命名生成。 将 `underscored `选项设置为 `true`，属性的 `field` 选项将被设置为属性名称的下划线版本。
3. `underscored` 将控制所有属性，包括时间戳，版本和外键。 它不会影响任何已经指定 `field` 选项的属性。

[#9304](https://github.com/sequelize/sequelize/pull/9304)

### 数据类型

**Postgres 范畴**

现在只支持一种标准格式 `[{ value: 1, inclusive: true }, { value: 20, inclusive: false }]` [#9364](https://github.com/sequelize/sequelize/pull/9364)

### 其它

Sequelize 现在对所有 INSERT / UPDATE操作（UPSERT除外）使用参数化查询。 它们可以更好地防范SQL注入攻击。

`ValidationErrorItem` 现在持有对 `original` 属性中原始错误的引用，而不是 `__raw` 属性。

[retry-as-promised](https://github.com/mickhansen/retry-as-promised) 已在 `3.0.0` 中更新, 它使用 [any-promise](https://github.com/kevinbeaty/any-promise). 这个模块重复所有 `sequelize.query` 操作。 您可以将 `any-promise` 配置为在 Node 4 或 6 上使用 `bluebird` 来获得更好的性能

### 已删除

- package: terraformer-wkt-parser [#9545](https://github.com/sequelize/sequelize/pull/9545)

## 更新日志

### 5.0.0-beta.8

- feat(query-generator): Generate INSERT / UPDATE using bind parameters [#9431](https://github.com/sequelize/sequelize/pull/9431) [#9492](https://github.com/sequelize/sequelize/pull/9492)
- performance: remove terraformer-wkt-parser dependency [#9545](https://github.com/sequelize/sequelize/pull/9545)
- fix(constructor): set username, password, database via options in addition to connection string[#9517](https://github.com/sequelize/sequelize/pull/9517)
- fix(associations/belongs-to-many): catch EmptyResultError in set/add helpers [#9535](https://github.com/sequelize/sequelize/pull/9535)
- fix: sync with alter:true doesn't use field name [#9529](https://github.com/sequelize/sequelize/pull/9529)
- fix(UnknownConstraintError): improper handling of error options [#9547](https://github.com/sequelize/sequelize/pull/9547)

### 5.0.0-beta.7

- fix(data-types/blob): only return null for mysql binary null [#9441](https://github.com/sequelize/sequelize/pull/9441)
- fix(errors): use standard .original rather than .__raw for actual error
- fix(connection-manager): mssql datatype parsing [#9470](https://github.com/sequelize/sequelize/pull/9470)
- fix(query/removeConstraint): support schemas
- fix: use Buffer.from
- fix(transactions): return patched promise from sequelize.query [#9473](https://github.com/sequelize/sequelize/pull/9473)

### 5.0.0-beta.6

- fix(postgres/query-generator): syntax error with auto-increment SMALLINT [#9406](https://github.com/sequelize/sequelize/pull/9406)
- fix(postgres/range): inclusive property lost in JSON format [#8471](https://github.com/sequelize/sequelize/issues/8471)
- fix(postgres/range): range bound not applied [#8176](https://github.com/sequelize/sequelize/issues/8176)
- fix(mssql): no unique constraint error thrown for PRIMARY case [#9415](https://github.com/sequelize/sequelize/pull/9415)
- fix(query-generator): regexp operator escaping
- docs: various improvements and hinting update

### 5.0.0-beta.5

- fix: inject foreignKey when using separate:true [#9396](https://github.com/sequelize/sequelize/pull/9396)
- fix(isSoftDeleted): just use deletedAt as flag
- feat(hasOne): sourceKey support with key validation [#9382](https://github.com/sequelize/sequelize/pull/9382)
- fix(query-generator/deleteQuery): remove auto limit [#9377](https://github.com/sequelize/sequelize/pull/9377)
- feat(postgres): skip locked support [#9197](https://github.com/sequelize/sequelize/pull/9197)
- fix(mssql): case sensitive operation fails because of uppercased system table references [#9337](https://github.com/sequelize/sequelize/pull/9337)

### 5.0.0-beta.4

- change(model): setDataValue should not mark null to null as changed [#9347](https://github.com/sequelize/sequelize/pull/9347)
- change(mysql/connection-manager): do not execute SET time_zone query if keepDefaultTimezone config is true [#9358](https://github.com/sequelize/sequelize/pull/9358)
- feat(transactions): Add afterCommit hooks for transactions [#9287](https://github.com/sequelize/sequelize/pull/9287)

### 5.0.0-beta.3

- change(model): new options.underscored implementation [#9304](https://github.com/sequelize/sequelize/pull/9304)
- fix(mssql): duplicate order generated with limit offset [#9307](https://github.com/sequelize/sequelize/pull/9307)
- fix(scope): do not assign scope on eagerly loaded associations [#9292](https://github.com/sequelize/sequelize/pull/9292)
- change(bulkCreate): only support non-empty array as updateOnDuplicate

### 5.0.0-beta.2

- change(operators): Symbol operators now enabled by default, removed deprecation warning
- fix(model): don't add LIMIT in findOne() queries on unique key [#9248](https://github.com/sequelize/sequelize/pull/9248)
- fix(model): use schema when generating foreign keys [#9029](https://github.com/sequelize/sequelize/issues/9029)

### 5.0.0-beta.1

- fix(postgres): reserved words support [#9236](https://github.com/sequelize/sequelize/pull/9236)
- fix(findOrCreate): warn and handle unknown attributes in defaults
- fix(query-generator): 1-to-many join in subQuery filter missing where clause [#9228](https://github.com/sequelize/sequelize/issues/9228)

### 5.0.0-beta

- `Model.attributes` now removed, use `Model.rawAttributes` [#5320](https://github.com/sequelize/sequelize/issues/5320)
- `paranoid` mode will now treat any record with `deletedAt` as deleted [#8496](https://github.com/sequelize/sequelize/issues/8496)
- Node 6 and up [#9015](https://github.com/sequelize/sequelize/issues/9015)
