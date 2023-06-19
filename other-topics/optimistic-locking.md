# Optimistic Locking - 乐观锁定

Sequelize 内置支持通过模型实例版本计数进行乐观锁定.

乐观锁定默认情况下处于禁用状态,可以通过在特定模型定义或全局模型配置中将 `version` 属性设置为 true 来启用.

有关更多详细信息,请参见[模型基础](../core-concepts/model-basics.md).

乐观锁定允许并发访问模型记录以进行编辑,并防止冲突覆盖数据. 它通过检查自从读取以来另一个进程是否对记录进行了更改,并在检测到冲突时抛出 OptimisticLockError 来执行此操作.