# Upgrade to V5 - 升级到 V5

Sequelize v5 是 v4 之后的下一个主要版本

### 突破性变化

### 支持 Node 6 以及更高版本

Sequelize v5 将仅支持 Node 6 以及更高版本 [#9015](https://github.com/sequelize/sequelize/issues/9015)

### 安全的运算符

在 v4 中,你会开始收到弃用警告 `String based operators are now deprecated (基于字符串的运算符现在已弃用)`. 同时也介绍了运算符的概念.这些运算符是一些防止散列注入攻击的符号.

详阅 `Querying - 查询`中的`运算符安全性` 内容

**对于 v5**

- 运算符现在默认启用.
- 你仍然可以通过在 `operatorsAliases` 中传递一个运算符映射来使用字符串运算符,但这会产生弃用警告.
- `Op.$raw` 已被移除.

### Typescript 支持

Sequelize 现在正式支持官方类型 [#10287](https://github.com/sequelize/sequelize/pull/10287). 你可以考虑远离可能不同步的外部类型.

### 池

对于 v5 Sequelize现在使用 `sequelize-pool`,它是 `generic-pool@2.5` 的现代化分支. 你不再需要调用 `sequelize.close` 来关闭池,这有助于 lambda 执行.[#8468](https://github.com/sequelize/sequelize/issues/8468).

### 模型

**验证器**

现在,当属性的值为 `null` 且 `allowNull` 为 `true` 时,运行每个属性定义的自定义验证器(与模型选项中定义的自定义验证器相对,之前它们没有运行且验证立即成功).为了避免升级时出现问题,请检查每个属性定义的所有自定义验证器,其中 `allowNull` 为 `true`,并确保当值为 `null` 时所有这些验证器都能正常运行. 参见 [#9143](https://github.com/sequelize/sequelize/issues/9143).

**属性**

`Model.attributes` 现在已被移除, 请使用 `Model.rawAttributes`. [#5320](https://github.com/sequelize/sequelize/issues/5320)

__注意__: _请不要将它与 `options.attributes` 混淆,它们仍然有效_

**偏执模式**

对于 v5 如果设置了 `deletedAt`, 该数据将被视为已删除. `paranoid` 选项只会使用 `deletedAt` 作为标志. [#8496](https://github.com/sequelize/sequelize/issues/8496)

**Model.bulkCreate**

`updateOnDuplicate` 选项用于接受布尔值和数组,现在只接受非空的数组属性. [#9288](https://github.com/sequelize/sequelize/issues/9288)


**下划线模式**

`Model.options.underscored` 的实现方式被改变了.你可以在 [这里](https://github.com/sequelize/sequelize/issues/6423#issuecomment-379472035) 找到完整的说明 .

大致内容

1. `underscoredAll` 和 `underscored` 选项都合并为一个 `underscored` 选项
2. 现在,所有属性都默认使用 camelcase 命名生成. 将 `underscored `选项设置为 `true`,属性的 `field` 选项将被设置为属性名称的下划线版本.
3. `underscored` 将控制所有属性,包括时间戳,版本和外键. 它不会影响任何已经指定 `field` 选项的属性.

[#9304](https://github.com/sequelize/sequelize/pull/9304)

**删除的别名**

许多基于模型的别名已被删除 [#9372](https://github.com/sequelize/sequelize/issues/9372)

| v5 删除 | 官方替代 |
| :------ | :------ |
| insertOrUpdate | upsert |
| find | findOne |
| findAndCount | findAndCountAll |
| findOrInitialize | findOrBuild |
| updateAttributes | update |
| findById, findByPrimary	| findByPk |
| all | findAll |
| hook | addHook |

### 数据类型

**范畴**

现在只支持一种标准格式 `[{ value: 1, inclusive: true }, { value: 20, inclusive: false }]` [#9364](https://github.com/sequelize/sequelize/pull/9364)

**不区分大小写的文本**

为 Postgres 和 SQLite 添加了对 `CITEXT` 的支持

**已删除**

`NONE` 类型已被删除,请使用 `VIRTUAL`

### Hooks

**删除的别名**

Hooks aliases has been removed [#9372](https://github.com/sequelize/sequelize/issues/9372)

| Rv5 删除 | 官方替代 |
| :------ | :------ |
| [after,before]BulkDelete | [after,before]BulkDestroy |
| [after,before]Delete | [after,before]Destroy |
| beforeConnection | beforeConnect |

### Sequelize

**删除的别名**

已删除许多常量,对象和类的原型引用 [#9372](https://github.com/sequelize/sequelize/issues/9372)

| v5 删除 | 官方替代 |
| :------ | :------ |
| Sequelize.prototype.Utils	| Sequelize.Utils	|
| Sequelize.prototype.Promise	| Sequelize.Promise	|
| Sequelize.prototype.TableHints | Sequelize.TableHints	|
| Sequelize.prototype.Op | Sequelize.Op	|
| Sequelize.prototype.Transaction	| Sequelize.Transaction	|
| Sequelize.prototype.Model	| Sequelize.Model	|
| Sequelize.prototype.Deferrable | Sequelize.Deferrable	|
| Sequelize.prototype.Error	| Sequelize.Error	|
| Sequelize.prototype[error] | Sequelize[error] |

```js
import Sequelize from 'sequelize';
const sequelize = new Sequelize('postgres://user:password@127.0.0.1:mydb');

/**
 * In v4 you can do this
 */
console.log(sequelize.Op === Sequelize.Op) // logs `true`
console.log(sequelize.UniqueConstraintError === Sequelize.UniqueConstraintError) // logs `true`

Model.findAll({
  where: {
    [sequelize.Op.and]: [ // Using sequelize.Op or Sequelize.Op interchangeably
      {
        name: "Abc"
      },
      {
        age: {
          [Sequelize.Op.gte]: 18
        }
      }
    ]
  }
}).catch(sequelize.ConnectionError, () => {
  console.error('Something wrong with connection?');
});

/**
 * In v5 aliases has been removed from Sequelize prototype
 * You should use Sequelize directly to access Op, Errors etc
 */

Model.findAll({
  where: {
    [Sequelize.Op.and]: [ // Dont use sequelize.Op, use Sequelize.Op instead
      {
        name: "Abc"
      },
      {
        age: {
          [Sequelize.Op.gte]: 18
        }
      }
    ]
  }
}).catch(Sequelize.ConnectionError, () => {
  console.error('Something wrong with connection?');
});
```

### 查询接口

- `changeColumn`不再使用`_idx`后缀生成约束. 现在Sequelize没有为约束指定任何名称,因此默认为数据库引擎命名. 这对齐了`sync`,`createTable`和`changeColumn`的行为.
- `addIndex` 别名的参数别名已被删除,请改用以下内容.
  - `indexName` => `name`
  - `indicesType` => `type`
  - `indexType`/`method` => `using`

### 其它

- Sequelize 现在对所有 INSERT / UPDATE操作(UPSERT除外)使用参数化查询. 它们可以更好地防范SQL注入攻击.

- `ValidationErrorItem` 现在持有对 `original` 属性中原始错误的引用,而不是 `__raw` 属性.

- [retry-as-promised](https://github.com/mickhansen/retry-as-promised) 已在 `3.1.0` 中更新, 它使用 [any-promise](https://github.com/kevinbeaty/any-promise). 这个模块重复所有 `sequelize.query` 操作. 你可以将 `any-promise` 配置为在 Node 4 或 6 上使用 `bluebird` 来获得更好的性能

- Sequelize将抛出`where`选项中的所有`undefined`键,在过去的版本中`undefined`被转换为`null`.

### 方言相关

#### MSSQL

- Sequelize 现在可以使用 `tedious >= 6.0.0`. 必须更新旧的 `dialectOptions` 以匹配其新格式. 请参阅 tedious [文档](http://tediousjs.github.io/tedious/api-connection.html#function_newConnection). 下面给出了一个新的 `dialectOptions` 的例子

```javascript
dialectOptions: {
  authentication: {
    domain: 'my-domain'
  },
  options: {
    requestTimeout: 60000,
    cryptoCredentialsDetails: {
      ciphers: "RC4-MD5"
    }
  }
}
```

#### MySQL

- 现有封装需要 `mysql2 >= 1.5.2`

#### MariaDB

- `dialect: 'mariadb'` 现在 [已支持](https://github.com/sequelize/sequelize/pull/10192) 使用 `mariadb` 包

### 包

- removed: terraformer-wkt-parser [#9545](https://github.com/sequelize/sequelize/pull/9545)
- removed: `generic-pool`
- added: `sequelize-pool`

## 更新日志

### 5.0.0-beta.17

- fix(build): default null for multiple primary keys
- fix(util): improve performance of classToInvokable [#10534](https://github.com/sequelize/sequelize/pull/10534)
- fix(model/update): propagate paranoid to individualHooks query [#10369](https://github.com/sequelize/sequelize/pull/10369)
- fix(association): use minimal select for hasAssociation [#10529](https://github.com/sequelize/sequelize/pull/10529)
- fix(query-interface): reject with error for describeTable [#10528](https://github.com/sequelize/sequelize/pull/10528)
- fix(model): throw for invalid include type [#10527](https://github.com/sequelize/sequelize/pull/10527)
- fix(types): additional options for db.query and add missing retry [#10512](https://github.com/sequelize/sequelize/pull/10512)
- fix(query): don't prepare options & sql for every retry [#10498](https://github.com/sequelize/sequelize/pull/10498)
- feat: expose Sequelize.BaseError
- feat: upgrade to tedious@6.0.0 [#10494](https://github.com/sequelize/sequelize/pull/10494)
- feat(sqlite/query-generator): support restart identity for truncate-table [#10522](https://github.com/sequelize/sequelize/pull/10522)
- feat(data-types): handle numbers passed as objects [#10492](https://github.com/sequelize/sequelize/pull/10492)
- feat(types): enabled string association [#10481](https://github.com/sequelize/sequelize/pull/10481)
- feat(postgres): allow customizing client_min_messages [#10448](https://github.com/sequelize/sequelize/pull/10448)
- refactor(data-types): move to classes [#10495](https://github.com/sequelize/sequelize/pull/10495)
- docs(legacy): fix N:M example [#10509](https://github.com/sequelize/sequelize/pull/10509)
- docs(migrations): use migrationStorageTableSchema [#10417](https://github.com/sequelize/sequelize/pull/10417)
- docs(hooks): add documentation for connection hooks [#10410](https://github.com/sequelize/sequelize/pull/10410)
- docs(addIndex): concurrently option [#10409](https://github.com/sequelize/sequelize/pull/10409)
- docs(model): fix typo [#10405](https://github.com/sequelize/sequelize/pull/10405)
- docs(usage): fix broken link on Basic Usage [#10381](https://github.com/sequelize/sequelize/pull/10381)
- docs(package.json): add homepage [#10372](https://github.com/sequelize/sequelize/pull/10372)

### 5.0.0-beta.16

- feat: add typescript typings [#10287](https://github.com/sequelize/sequelize/pull/10117)
- fix(mysql): match with newlines in error message [#10320](https://github.com/sequelize/sequelize/pull/10320)
- fix(update): skips update when nothing to update [#10248](https://github.com/sequelize/sequelize/pull/10248)
- fix(utils): flattenObject for null values [#10293](https://github.com/sequelize/sequelize/pull/10293)
- fix(instance-validator): don't skip custom validators on null [#9143](https://github.com/sequelize/sequelize/pull/9143)
- docs(transaction): after save example [#10280](https://github.com/sequelize/sequelize/pull/10280)
- docs(query-generator): typo [#10277](https://github.com/sequelize/sequelize/pull/10277)
- refactor(errors): restructure [#10355](https://github.com/sequelize/sequelize/pull/10355)
- refactor(scope): documentation #9087 [#10312](https://github.com/sequelize/sequelize/pull/10312)
- refactor: cleanup association and spread use [#10276](https://github.com/sequelize/sequelize/pull/10276)

### 5.0.0-beta.15

- fix(query-generator): fix addColumn create comment [#10117](https://github.com/sequelize/sequelize/pull/10117)
- fix(sync): throw when no models defined [#10175](https://github.com/sequelize/sequelize/pull/10175)
- fix(association): enable eager load with include all(#9928) [#10173](https://github.com/sequelize/sequelize/pull/10173)
- fix(sqlite): simplify connection error handling
- fix(model): prevent version number from being incremented as string [#10217](https://github.com/sequelize/sequelize/pull/10217)
- feat(dialect): mariadb [#10192](https://github.com/sequelize/sequelize/pull/10192)
- docs(migrations): improve dialect options docs
- docs: fix favicon [#10242](https://github.com/sequelize/sequelize/pull/10242)
- docs(model.init): `attribute.column.validate` option [#10237](https://github.com/sequelize/sequelize/pull/10237)
- docs(bulk-create): update support information about ignoreDuplicates
- docs: explain custom/new data types [#10170](https://github.com/sequelize/sequelize/pull/10170)
- docs(migrations): Simplify CLI Call [#10201](https://github.com/sequelize/sequelize/pull/10201)
- docs(migrations): added advanced skeleton example [#10190](https://github.com/sequelize/sequelize/pull/10190)
- docs(transaction): default isolation level [#10111](https://github.com/sequelize/sequelize/pull/10111)
- docs: typo in associations.md [#10157](https://github.com/sequelize/sequelize/pull/10157)
- refactor: reduce code complexity [#10120](https://github.com/sequelize/sequelize/pull/10120)
- refactor: optimize memoize use, misc cases [#10122](https://github.com/sequelize/sequelize/pull/10122)
- chore(lint): enforce consistent spacing [#10193](https://github.com/sequelize/sequelize/pull/10193)



### 5.0.0-beta.14

- fix(query): correctly quote identifier for attributes (#9964) [#10118](https://github.com/sequelize/sequelize/pull/10118)
- feat(postgres): dyanmic oids [#10077](https://github.com/sequelize/sequelize/pull/10077)
- fix(error): optimistic lock message [#10068](https://github.com/sequelize/sequelize/pull/10068)
- fix(package): update depd to version 2.0.0 [#10081](https://github.com/sequelize/sequelize/pull/10081)
- fix(model): validate virtual attribute (#9947) [#10085](https://github.com/sequelize/sequelize/pull/10085)
- fix(test): actually test get method with raw option [#10059](https://github.com/sequelize/sequelize/pull/10059)
- fix(model): return deep cloned value for toJSON [#10058](https://github.com/sequelize/sequelize/pull/10058)
- fix(model): create instance with many-to-many association with extra column (#10034) [#10050](https://github.com/sequelize/sequelize/pull/10050)
- fix(query-generator): fix bad property access [#10056](https://github.com/sequelize/sequelize/pull/10056)
- docs(upgrade-to-v4): typo [#10060](https://github.com/sequelize/sequelize/pull/10060)
- docs(model-usage): order expression format [#10061](https://github.com/sequelize/sequelize/pull/10061)
- chore(package): update retry-as-promised to version 3.1.0 [#10065](https://github.com/sequelize/sequelize/pull/10065)
- refactor(scopes): just in time options conforming [#9735](https://github.com/sequelize/sequelize/pull/9735)
- refactor: use sequelize-pool for pooling [#10051](https://github.com/sequelize/sequelize/pull/10051)
- refactor(*): cleanup code [#10091](https://github.com/sequelize/sequelize/pull/10091)
- refactor: use template strings [#10055](https://github.com/sequelize/sequelize/pull/10055)
- refactor(query-generation): cleanup template usage [#10047](https://github.com/sequelize/sequelize/pull/10047)

### 5.0.0-beta.13

- fix: throw on undefined where parameters [#10048](https://github.com/sequelize/sequelize/pull/10048)
- fix(model): improve wrong alias error message [#10041](https://github.com/sequelize/sequelize/pull/10041)
- feat(sqlite): CITEXT datatype [#10036](https://github.com/sequelize/sequelize/pull/10036)
-  fix(postgres): remove if not exists and cascade from create/drop database queries [#10033](https://github.com/sequelize/sequelize/pull/10033)
- fix(syntax): correct parentheses around union [#10003](https://github.com/sequelize/sequelize/pull/10003)
- feat(query-interface): createDatabase / dropDatabase support [#10027](https://github.com/sequelize/sequelize/pull/10027)
- feat(postgres): CITEXT datatype [#10024](https://github.com/sequelize/sequelize/pull/10024)
- feat: pass uri query parameters to dialectOptions [#10025](https://github.com/sequelize/sequelize/pull/10025)
- docs(query-generator): remove doc about where raw query [#10017](https://github.com/sequelize/sequelize/pull/10017)
- fix(query): handle undefined field on unique constraint error [#10018](https://github.com/sequelize/sequelize/pull/10018)
- fix(model): sum returns zero when empty matching [#9984](https://github.com/sequelize/sequelize/pull/9984)
- feat(query-generator): add startsWith, endsWith and substring operators [#9999](https://github.com/sequelize/sequelize/pull/9999)
- docs(sequelize): correct jsdoc annotations for authenticate [#10002](https://github.com/sequelize/sequelize/pull/10002)
- docs(query-interface): add bulkUpdate docs [#10005](https://github.com/sequelize/sequelize/pull/10005)
- fix(tinyint): ignore params for TINYINT on postgres [#9992](https://github.com/sequelize/sequelize/pull/9992)
- fix(belongs-to): create now returns target model [#9980](https://github.com/sequelize/sequelize/pull/9980)
- refactor(model): remove .all alias [#9975](https://github.com/sequelize/sequelize/pull/9975)
- perf: fix memory leak due to instance reference by isImmutable [#9973](https://github.com/sequelize/sequelize/pull/9973)
- feat(sequelize): dialectModule option [#9972](https://github.com/sequelize/sequelize/pull/9972)
- fix(query): check valid warn message [#9948](https://github.com/sequelize/sequelize/pull/9948)
- fix(model): check for own property when overriding association mixins [#9953](https://github.com/sequelize/sequelize/pull/9953)
- fix(create-table): support for uniqueKeys [#9946](https://github.com/sequelize/sequelize/pull/9946)
- refactor(transaction): remove autocommit mode [#9921](https://github.com/sequelize/sequelize/pull/9921)
- feat(sequelize): getDatabaseName [#9937](https://github.com/sequelize/sequelize/pull/9937)
- refactor: remove aliases [#9933](https://github.com/sequelize/sequelize/pull/9933)
- feat(belongsToMany): override unique constraint name with uniqueKey [#9914](https://github.com/sequelize/sequelize/pull/9914)
- fix(postgres): properly disconnect connections [#9911](https://github.com/sequelize/sequelize/pull/9911)
- docs(instances.md): add section for restore() [#9917](https://github.com/sequelize/sequelize/pull/9917)
- docs(hooks.md): add warning about memory limits of individual hooks [#9881](https://github.com/sequelize/sequelize/pull/9881)
- fix(package): update debug to version 4.0.0 [#9908](https://github.com/sequelize/sequelize/pull/9908)
- feat(postgres): support ignoreDuplicates with ON CONFLICT DO NOTHING [#9883](https://github.com/sequelize/sequelize/pull/9883)

### 5.0.0-beta.12

- fix(changeColumn): normalize attribute [#9897](https://github.com/sequelize/sequelize/pull/9897)
- feat(describeTable): support string length for mssql [#9896](https://github.com/sequelize/sequelize/pull/9896)
- feat(describeTable): support autoIncrement for mysql [#9894](https://github.com/sequelize/sequelize/pull/9894)
- fix(sqlite): unable to reference foreignKey on primaryKey [#9893](https://github.com/sequelize/sequelize/pull/9893)
- fix(postgres): enum with string COMMENT breaks query [#9891](https://github.com/sequelize/sequelize/pull/9891)
- fix(changeColumn): use engine defaults for foreign/unique key naming [#9890](https://github.com/sequelize/sequelize/pull/9890)
- fix(transaction): fixed unhandled rejection when connection acquire timeout [#9879](https://github.com/sequelize/sequelize/pull/9879)
- fix(sqlite): close connection properly and cleanup files [#9851](https://github.com/sequelize/sequelize/pull/9851)
- fix(model): incorrect error message for findCreateFind [#9849](https://github.com/sequelize/sequelize/pull/9849)

### 5.0.0-beta.11

- fix(count): duplicate mapping of fields break scopes [#9788](https://github.com/sequelize/sequelize/pull/9788)
- fix(model): bulkCreate should populate dataValues directly [#9797](https://github.com/sequelize/sequelize/pull/9797)
- fix(mysql): improve unique key violation handling [#9724](https://github.com/sequelize/sequelize/pull/9724)
- fix(separate): don't propagate group to separated queries [#9754](https://github.com/sequelize/sequelize/pull/9754)
- fix(scope): incorrect query generated when sequelize.fn used with scopes [#9730](https://github.com/sequelize/sequelize/pull/9730)
- fix(json): access included data with attributes [#9662](https://github.com/sequelize/sequelize/pull/9662)
- (fix): pass offset in UNION'ed queries [#9577](https://github.com/sequelize/sequelize/pull/9577)
- fix(destroy): attributes updated in a beforeDestroy hook are now persisted on soft delete [#9319](https://github.com/sequelize/sequelize/pull/9319)
- fix(addScope): only throw when defaultScope is defined [#9703](https://github.com/sequelize/sequelize/pull/9703)


### 5.0.0-beta.10

- fix(belongsToMany): association.add returns array of array of through records [#9700](https://github.com/sequelize/sequelize/pull/9700)
- feat: association hooks [#9590](https://github.com/sequelize/sequelize/pull/9590)
- fix(bulkCreate): dont map dataValue to fields for individualHooks:true[#9672](https://github.com/sequelize/sequelize/pull/9672)
- feat(postgres): drop enum support [#9641](https://github.com/sequelize/sequelize/pull/9641)
- feat(validation): improve validation for type[#9660](https://github.com/sequelize/sequelize/pull/9660)
- feat: allow querying sqlite_master table [#9645](https://github.com/sequelize/sequelize/pull/9645)
- fix(hasOne.sourceKey): setup sourceKeyAttribute for joins [#9658](https://github.com/sequelize/sequelize/pull/9658)
- fix: throw when type of array values is not defined [#9649](https://github.com/sequelize/sequelize/pull/9649)
- fix(query-generator): ignore undefined keys in query [#9548](https://github.com/sequelize/sequelize/pull/9548)
- fix(model): unable to override rejectOnEmpty [#9632](https://github.com/sequelize/sequelize/pull/9632)
- fix(reload): instance.changed() remains unaffected [#9615](https://github.com/sequelize/sequelize/pull/9615)
- feat(model): column level comments [#9573](https://github.com/sequelize/sequelize/pull/9573)
- docs: cleanup / correct jsdoc references [#9702](https://github.com/sequelize/sequelize/pull/9702)


### 5.0.0-beta.9

- fix(model): ignore undefined values in update payload [#9587](https://github.com/sequelize/sequelize/pull/9587)
- fix(mssql): set encrypt as default false for dialect options [#9588](https://github.com/sequelize/sequelize/pull/9588)
- fix(model): ignore VIRTUAL/getters with attributes.exclude [#9568](https://github.com/sequelize/sequelize/pull/9568)
- feat(data-types): CIDR, INET, MACADDR support for Postgres [#9567](https://github.com/sequelize/sequelize/pull/9567)
- fix: customize allowNull message with notNull validator [#9549](https://github.com/sequelize/sequelize/pull/9549)

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
