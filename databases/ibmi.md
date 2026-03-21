# Sequelize 用于 DB2 for IBM i



我们对 DB2 for IBM i 的实现并未在真实数据库上进行集成测试。
因此，我们无法保证其能如预期工作，也无法保证其稳定性。

我们依赖社区的帮助来改进此方言。



请参阅 [Releases](/releases#db2-for-ibm-i-support-table) 以了解支持哪些版本的 DB2 for IBM i。


要在 DB2 for IBM i 上使用 Sequelize，你需要安装 `@sequelize/db2-ibmi` 方言包：

```bash npm2yarn
npm i @sequelize/db2-ibmi
```


然后在 Sequelize 构造函数中使用 `IbmiDialect` 作为 dialect 选项：

```ts
import { Sequelize } from '@sequelize/core';
import { IbmiDialect } from '@sequelize/db2-ibmi';

const sequelize = new Sequelize({
  dialect: IbmiDialect,
  odbcConnectionString: 'DSN=MYDSN;UID=myuser;PWD=mypassword',
  connectionTimeout: 60,
});
```


## 连接选项

import ConnectionOptions from './_connection-options.md';

<ConnectionOptions />


DB2 for IBM i 方言支持以下选项：

| 选项                    | 说明                                                                                                          |
| ---------------------- | ------------------------------------------------------------------------------------------------------------- |
| `connectionTimeout`    | 等待连接上请求完成后返回应用的秒数                                                                            |
| `loginTimeout`         | 等待登录请求完成后返回应用的秒数                                                                              |
| `odbcConnectionString` | 用于连接数据库的连接字符串。如果提供此项，下方选项可省略。                                                    |
| `dataSourceName`       | 连接字符串中的 ODBC "DSN" 部分。                                                                             |
| `username`             | 连接字符串中的 ODBC "UID" 部分。                                                                             |
| `system`               | 连接字符串中的 ODBC "SYSTEM" 部分。                                                                          |
| `password`             | 连接字符串中的 ODBC "PWD" 部分。                                                                             |
