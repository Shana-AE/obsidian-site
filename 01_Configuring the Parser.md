You can use `#[command]` attributes on the struct to change the application level behavior of clap. Any [`Command`](https://docs.rs/clap/latest/clap/struct.Command.html "struct clap::Command") builder function can be used as an attribute, like [`Command::next_line_help`](https://docs.rs/clap/latest/clap/struct.Command.html#method.next_line_help "method clap::Command::next_line_help").

`#[command]` attribute就是Command的builder function