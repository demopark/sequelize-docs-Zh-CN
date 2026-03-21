连接选项用于配置与数据库的连接。

最简单的用法是在配置对象的根部直接使用这些选项。这些选项也可以在 [`replication`](../other-topics/read-replication.md) 选项中使用，以自定义每个副本的连接，
并且可以通过 [`beforeConnect`](../other-topics/hooks.mdx) 钩子按连接逐一修改。
