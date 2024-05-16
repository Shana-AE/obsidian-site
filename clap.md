[clap]([clap - Rust (docs.rs)](https://docs.rs/clap/latest/clap/index.html))

see also: 
- [FAQ: When should I use the builder vs derive APIs?](https://docs.rs/clap/latest/clap/_faq/index.html#when-should-i-use-the-builder-vs-derive-apis "mod clap::_faq")

## 文档中不好找的

- default_value = "xxx" 如果实现了From trait会将xxx转换为定义的类型，比如下边会将"output.json"转换为String，即`"output.json".into()`
  ```rust
      struct Option {
        #[arg(default_value="output.json")]
        output: String,
      }
	```

文档注释应该会变为cli中的提示

subcommand上也可以加入`#[command()]`
````ad-example
title: subcommand 的 command attribute
```rust
enum SubCommand {
    #[command(name="csv", about="Show CSV, or convert CSV to other formats")] // name为subcommand的实际使用
    Csv(CsvOpts),
}
```

输出
```log
Usage: rcli <COMMAND>

Commands:
  csv   Show CSV, or convert CSV to other formats
  help  Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version       
```
````