# Read Replication - 读取复制

Sequelize 支持 [读取复制](https://en.wikipedia.org/wiki/Replication_%28computing%29#Database_replication), 即,当你要执行 SELECT 查询时,可以连接多个服务器. 当你执行读取复制时,你可以指定一台或多台服务器充当读取副本,并指定一台服务器充当写入主机,该主机处理所有写入和更新并将它们传播到副本(请注意,实际复制过程 *不是* 由 Sequelize 处理,而应由数据库后端设置).

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mysql',
  port: 3306,
  replication: {
    read: [
      { host: '8.8.8.8', username: 'read-1-username', password: process.env.READ_DB_1_PW },
      { host: '9.9.9.9', username: 'read-2-username', password: process.env.READ_DB_2_PW }
    ],
    write: { host: '1.1.1.1', username: 'write-username', password: process.env.WRITE_DB_PW }
  },
  pool: { // 如果要覆盖用于 读/写 池的参数,可以在此处执行
    max: 20,
    idle: 30000
  },
})
```

如果你有适用于所有副本的常规设置,则无需为每个实例提供它们. 在上面的代码中,数据库名称和端口将传播到所有副本. 如果将用户名和密码留给任何副本,则同样会生效. 每个副本具有以下参数：`host`, `port`, `username`, `password`, `database`.

Sequelize 使用池来管理与副本的连接. 在内部,Sequelize 将维护使用 `pool` 配置创建的两个池.

如果要修改这些设置,可以在实例化 Sequelize 时将 pool 作为参数传递,如上所示.

每个 `write` 或 `useMaster: true` 查询都将使用写入池. 对于 `SELECT`,将使用读取池. 使用基本的循环调度来切换只读副本.