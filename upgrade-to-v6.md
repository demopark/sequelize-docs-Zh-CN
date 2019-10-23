# Upgrade to V6 - 升级到 V6

Sequelize v6 是 v5 之后的下一个主要版本

## 突破性变化

### 支持 Node 10 以及更高版本

Sequelize v6 将仅支持 Node 10 以及更高版本 [#9015](https://github.com/sequelize/sequelize/issues/9015)

### CLS

你现在应该使用 [cls-hooked](https://github.com/Jeff-Lewis/cls-hooked) 包来支持 CLS.

```js
  const cls = require('cls-hooked');
  const namespace = cls.createNamespace('....');
  const Sequelize = require('sequelize');

  Sequelize.useCLS(namespace);
```

Bluebird [现在支持](https://github.com/petkaantonov/bluebird/issues/1403) `async_hooks`. 调用 `Sequelize.useCLS` 时将自动启用此配置. 因此，所有的 promises 都应保持 CLS 上下文，而不要对 `cls-bluebird` 进行修补.

### 模型

**`options.returning`**

参数 `returning: true` 将不再返回模型中未定义的属性. 可以使用 `returning: ['*']` 恢复之前的作用。

**`Model.changed()`**

现在，此方法使用 `_.isEqual` 来测试是否相等。 修改 JSON 对象的嵌套值不会将它们标记为已更改，因为它仍然是同一对象。

```js
  const instance = await MyModel.findOne();

  instance.myJsonField.a = 1;
  console.log(instance.changed()) => false

  await instance.save(); // 这不会保存任何东西

  instance.changed('myJsonField', true);
  console.log(instance.changed()) => ['myJsonField']

  await instance.save(); // 将会保存
```

## 更新日志

### 6.0.0-beta.3

- feat: support cls-hooked / tests [#11584](https://github.com/sequelize/sequelize/pull/11584)

### 6.0.0-beta.2

- feat(postgres): change returning option to only return model attributes [#11526](https://github.com/sequelize/sequelize/pull/11526)
- fix(associations): allow binary key for belongs-to-many [#11578](https://github.com/sequelize/sequelize/pull/11578)
- fix(postgres): always replace returning statement for upsertQuery
- fix(model): make .changed() deep aware [#10851](https://github.com/sequelize/sequelize/pull/10851)
- change: use node 10 [#11580](https://github.com/sequelize/sequelize/pull/11580)
