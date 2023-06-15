# Connection Pool - 连接池

如果要从单个进程连接到数据库,则应仅创建一个 Sequelize 实例. Sequelize 将在初始化时建立连接池. 可以通过构造函数的 `options` 参数(使用 `options.pool`)来配置此连接池,如以下示例所示：

```js
const sequelize = new Sequelize(/* ... */, {
  // ...
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});
```

在 [Sequelize 构造函数的 API 参考](https://sequelize.org/api/v6/class/src/sequelize.js~Sequelize.html#instance-constructor-constructor)中了解更多信息. 如果要从多个进程连接到数据库,则必须为每个进程创建一个实例,但是每个实例的最大连接池大小应达到最大总大小. 例如,如果你希望最大连接池大小为90,并且有三个进程,则每个进程的 Sequelize 实例的最大连接池大小应为30.
