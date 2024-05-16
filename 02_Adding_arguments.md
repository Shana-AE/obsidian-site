Note that the [default `ArgAction` is `Set`](https://docs.rs/clap/latest/clap/_derive/_tutorial/index.html#arg-types "mod clap::_derive::_tutorial"). To accept multiple values, override the [action](https://docs.rs/clap/latest/clap/struct.Arg.html#method.action "method clap::Arg::action") with [`Append`](https://docs.rs/clap/latest/clap/enum.ArgAction.html#variant.Append "variant clap::ArgAction::Append") via `Vec`:

Subcommands are derived with `#[derive(Subcommand)]` and be added via [`#[command(subcommand)]` attribute](https://docs.rs/clap/latest/clap/_derive/_tutorial/index.html#command-attributes "mod clap::_derive::_tutorial") on the field using that type. Each instance of a [Subcommand](https://docs.rs/clap/latest/clap/trait.Subcommand.html "trait clap::Subcommand") can have its own version, author(s), Args, and even its own subcommands.

可以用struct变体或struct
We used a struct-variant to define the `add` subcommand. Alternatively, you can use a struct for your subcommand’s arguments:

When specifying commands with `command: Commands`, they are required. Alternatively, you could do `command: Option<Commands>` to make it optional.