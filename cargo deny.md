分享一个个人的情况

问题如下：
failed to open advisory database: "$CARGO_HOME/advisory-db/github.com-2f857891b7f43c59" does not appear to be a git repository: Could not retrieve metadata of "$CARGO_HOME/advisory-db/github.com-2f857891b7f43c59": No such file or directory (os error 2)

检查 $CARGO_HOME/advisory-db 下面只有一个 db.lock 文件

查代码发现 cargo-deny 依靠 DbSet 的 load 命令拉取 db-urls 的东西，但是 cargo deny init 不会执行拉取逻辑

解决方案：cargo-deny 有个文档没写的命令，叫 fetch，cargo deny fetch 一下就好了