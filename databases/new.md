# 其他数据库


如果你正在使用 Sequelize 当前尚未支持的数据库，你有以下几种选择：

- [请求支持新的方言](#requesting-support-for-a-new-dialect)
- [贡献新的方言](#contributing-a-new-dialect)
- [创建第三方方言](#creating-a-third-party-dialect)


## 请求支持新的方言


你可以在 [Sequelize 仓库](https://github.com/sequelize/sequelize) 提交功能请求，申请支持新的方言。
请确保你请求的方言没有已有的 issue。[在这里查看所有新方言的请求列表](https://github.com/sequelize/sequelize/issues?q=is%3Aopen+is%3Aissue+label%3A%22dialect%3A+new%22)。


我们对接受新方言有如下条件：

- 数据库必须支持 SQL 查询。
- 数据库必须有可用于连接的 Node.js 库。
- 我们必须能在 GitHub Codespaces 中运行数据库实例，以保证本地开发可用。
- 我们必须能在 GitHub Actions 中运行数据库实例，以便集成测试。
- 集成测试必须能在 15 分钟内完成。


你也可以通过赞助的方式推动新方言的开发。如果你对此感兴趣，[请通过邮件联系我们](https://github.com/sequelize/sequelize/blob/main/CONTACT.md)。
请注意，上述条件同样适用于赞助方言，并且实现新方言可能需要较大投入。


如果该方言符合要求，我们会在有足够用户请求或获得赞助时考虑实现。
个人贡献者也可以选择[贡献新的方言](#contributing-a-new-dialect)。


## 贡献新的方言



我们不再接受 Sequelize 6 的新方言。


如果你希望通过 pull request 添加新方言，请遵循以下步骤：

- 确保该方言符合 [纳入要求](#requesting-support-for-a-new-dialect)。
- [为新方言提交功能请求](#requesting-support-for-a-new-dialect)，如果还没有的话。
- 在功能请求中注明你正在实现该方言（即使不是你自己发起的请求）。
- 阅读现有方言包的源码以了解其工作方式。目前我们没有创建新方言的官方指南。


## 创建第三方方言



虽然我们正在逐步提供用于创建第三方方言的稳定公共 API，但许多必要的 API 仍为内部实现，且可能会变动。

如果你实现了第三方方言，请注意它可能会在 Sequelize 的后续非主版本中失效。


如果你的方言不符合纳入 Sequelize 的要求，你可以选择创建第三方方言。

Each dialect is a separate package that extends Sequelize with the necessary functionality to connect to the database.
Third-party dialects can be published on npm and used in your project like any other package.

Consider exploring the source code of an existing dialect package to understand how it works.
We unfortunately do not have a guide for creating a new dialect at this time.
