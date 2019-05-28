# Read replication - 读取复制

Sequelize 支持读复制,即当你想要进行 SELECT 查询时,可以连接多个服务器. 执行读取复制时,指定一个或多个服务器作为只读副本,并且一个服务器替换相同的副本(请注意,Sequelize**不会**处理实际的复制过程,而应由后端数据库处理).

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mysql',
  port: 3306
  replication: {
    read: [
      { host: '8.8.8.8', username: 'read-username', password: 'some-password' },
      { host: '9.9.9.9', username: 'another-username', password: null }
    ],
    write: { host: '1.1.1.1', username: 'write-username', password: 'any-password' }
  },
  pool: { // 如果要覆盖用于读/写池的参数,可以在此处执行此操作
    max: 20,
    idle: 30000
  },
})
```
如果你有适用于每个实例的任何常规设置. 在上面的代码中,数据库名称和端口将传播到所有副本. 对于用户和密码,如果你将其留给任何副本,也会发生同样的情况.每个副本都有以下参数:`host`,`port`,`username`,`password`,`database`.

Sequelize 使用池来管理与副本的连接. 内部 Sequelize 将维护使用 `pool` 配置创建的两个池.

如果要修改这些,可以在实例化 Sequelize 时将池作为参数传递,如上所示.

每个 `write` 或 `useMaster: true` 查询都将使用写入池. 对于 `SELECT`,将使用读取池. 使用基本循环调度切换只读副本.