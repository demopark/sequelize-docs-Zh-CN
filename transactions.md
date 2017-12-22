# Transactions - 事务

Sequelize 支持两种使用事务的方法：

* 一个将根据 promise 链的结果自动提交或回滚事务，（如果启用）用回调将该事务传递给所有调用
* 而另一个 leave committing，回滚并将事务传递给用户。

主要区别在于托管事务使用一个回调，对非托管事务而言期望 promise 返回一个 promise 的结果。

## 托管事务（auto-callback）

托管事务自动处理提交或回滚事务。你可以通过将回调传递给 `sequelize.transaction` 来启动托管事务。

注意回传传递给 `transaction` 的回调是否是一个 promise 链，并且没有明确地调用`t.commit（）`或  `t.rollback()`。 如果返回链中的所有 promise 都已成功解决，则事务被提交。 如果一个或几个 promise 被拒绝，事务将回滚。

```js
return sequelize.transaction(function (t) {

  // 在这里链接您的所有查询。 确保你返回他们。
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(function (user) {
    return user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, {transaction: t});
  });

}).then(function (result) {
  // 事务已被提交
  // result 是 promise 链返回到事务回调的结果
}).catch(function (err) {
  // 事务已被回滚
  // err 是拒绝 promise 链返回到事务回调的错误
});
```

### 抛出错误到回滚

使用托管事务时，你应该 _**永不**_ 手动提交或回滚事务。 如果所有查询都成功，但您仍然希望回滚事务（例如因为验证失败），则应该抛出一个错误来断开和拒绝链接：

```js
return sequelize.transaction(function (t) {
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(function (user) {
    // 查询成功，但我们仍然想回滚！
    throw new Error();
  });
});
```

### 自动将事务传递给所有查询

在上面的例子中，事务仍然是手动传递的，通过传递 `{transaction:t}` 作为第二个参数。 要自动将事务传递给所有查询，您必须安装 [continuation local storage](https://github.com/othiym23/node-continuation-local-storage) (CLS) 模块，并在您自己的代码中实例化一个命名空间：

```js
const cls = require('continuation-local-storage'),
    namespace = cls.createNamespace('my-very-own-namespace');
```

要启用CLS，您必须通过使用sequelize构造函数的静态方法来告诉Sequelize要使用的命名空间：

```js
const Sequelize = require('sequelize');
Sequelize.useCLS(namespace);

new Sequelize(....);
```

请注意， `useCLS()`  方法在 *构造函数* 上，而不是在 sequelize 的实例上。 这意味着所有实例将共享相同的命名空间，并且 CLS 是全部或全无方式 - 你不能仅在某些实例中启用它。

CLS 的工作方式就像一个用于回调的本地线程存储。 这在实践中意味着不同的回调链可以通过使用 CLS 命名空间来访问局部变量。 当启用 CLS 时，创建新事务时，Sequelize 将在命名空间上设置 `transaction` 属性。 由于回调链中设置的变量对该链是私有的，因此可以同时存在多个并发事务：

```js
sequelize.transaction(function (t1) {
  namespace.get('transaction') === t1; // true
});

sequelize.transaction(function (t2) {
  namespace.get('transaction') === t2; // true
});
```

在大多数情况下，你不需要直接访问 `namespace.get('transaction')`，因为所有查询都将自动在命名空间中查找事务：

```js
sequelize.transaction(function (t1) {
  // 启用 CLS 后，将在事务中创建用户
  return User.create({ name: 'Alice' });
});
```

在使用 `Sequelize.useCLS()` 后，从 sequelize 返回的所有 promise 将被修补以维护 CLS 上下文。 CLS 是一个复杂的课题 - [cls-bluebird](https://www.npmjs.com/package/cls-bluebird)的文档中有更多细节，用于使 bluebird promise 的补丁与CLS一起工作。

**注意:** _[当使用 cls-hooked 软件包的时候，CLS 仅支持 async/await](https://github.com/othiym23/node-continuation-local-storage/issues/98#issuecomment-323503807). 即使, [cls-hooked](https://github.com/Jeff-Lewis/cls-hooked/blob/master/README.md) 依靠 *试验性的 API* [async_hooks](https://github.com/nodejs/node/blob/master/doc/api/async_hooks.md)_

## 并行/部分事务

你可以在一系列查询中执行并发事务，或者将某些事务从任何事务中排除。 使用 `{transaction: }` 选项来控制查询所属的事务：

**警告:** _SQLite 不能同时支持多个事务。_

### 不启用CLS

```js
sequelize.transaction(function (t1) {
  return sequelize.transaction(function (t2) {
    // 启用CLS，这里的查询将默认使用 t2    
    // 通过 `transaction` 选项来定义/更改它们所属的事务。
        return Promise.all([
        User.create({ name: 'Bob' }, { transaction: null }),
        User.create({ name: 'Mallory' }, { transaction: t1 }),
        User.create({ name: 'John' }) // 这将默认为 t2
    ]);
  });
});
```

## 隔离等级

启动事务时可能使用的隔离等级：

```js
Sequelize.Transaction.ISOLATION_LEVELS.READ_UNCOMMITTED // "READ UNCOMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.READ_COMMITTED // "READ COMMITTED"
Sequelize.Transaction.ISOLATION_LEVELS.REPEATABLE_READ  // "REPEATABLE READ"
Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE // "SERIALIZABLE"
```

默认情况下，sequelize 使用数据库的隔离级别。 如果要使用不同的隔离级别，请传入所需级别作为第一个参数：

```js
return sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
  }, function (t) {

  //  你的事务

  });
```

**注意:** _在MSSQL的情况下，SET ISOLATION LEVEL 查询不被记录, 指定的 isolationLevel 直接传递到 tedious_

## 非托管事务（then-callback）

非托管事务强制您手动回滚或提交交易。 如果不这样做，事务将挂起，直到超时。 要启动非托管事务，请调用 `sequelize.transaction()` 而不用 callback（你仍然可以传递一个选项对象），并在返回的 promise 上调用 `then`。 请注意，`commit()` 和 `rollback()` 返回一个 promise。

```js
return sequelize.transaction().then(function (t) {
  return User.create({
    firstName: 'Bart',
    lastName: 'Simpson'
  }, {transaction: t}).then(function (user) {
    return user.addSibling({
      firstName: 'Lisa',
      lastName: 'Simpson'
    }, {transaction: t});
  }).then(function () {
    return t.commit();
  }).catch(function (err) {
    return t.rollback();
  });
});
```

## 参数

可以使用options对象作为第一个参数来调用`transaction`方法，这允许配置事务。

```js
return sequelize.transaction({ /* options */ });
```

以下选项（使用默认值）可用：

```js
{
  autocommit: true,
  isolationLevel: 'REPEATABLE_READ',
  deferrable: 'NOT DEFERRABLE' // postgres 的默认设置
}
```

在为 Sequelize 实例或每个局部事务初始化时，`isolationLevel`可以全局设置：

```js
// 全局
new Sequelize('db', 'user', 'pw', {
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});

// 局部
sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
```

`deferrable` 选项在事务开始后触发一个额外的查询，可选地将约束检查设置为延迟或立即。 请注意，这仅在PostgreSQL中受支持。

```js
sequelize.transaction({
  // 推迟所有约束：
  deferrable: Sequelize.Deferrable.SET_DEFERRED,

  // 推迟具体约束：
  deferrable: Sequelize.Deferrable.SET_DEFERRED(['some_constraint']),

  // 不推迟约束：
  deferrable: Sequelize.Deferrable.SET_IMMEDIATE
})
```

## 使用其他 Sequelize 方法

`transaction` 选项与其他大多数选项一起使用，通常是方法的第一个参数。
对于取值的方法，如 `.create`, `.update()`, `.updateAttributes()` 等。应该传递给第二个参数的选项。
如果不确定，请参阅API文档中的用于确定签名的方法。
