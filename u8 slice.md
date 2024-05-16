```rust
fn try_new(key: &[u8]) -> anyhow::Result<Self> {
	let key = &key[..32]; // key 为 &[u8]
```

在Rust中，当你使用 `&` 符号获取一个变量的引用时，你会得到一个指向该变量的引用。然而，当你对一个已经是引用的变量再次使用 `&` 符号时，你会得到一个指向引用的引用，也就是 `&&`。

在你的代码中，[`key`](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/process/text.rs") 变量原本就是一个引用，所以你可能期望 `&key[..32]` 的结果是 `&&[u8]`。但实际上，这里的 `&` 符号并不是在获取 [`key`](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/process/text.rs") 的引用，而是在获取 [`key[..32]`](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/process/text.rs") 的引用。`[..32]` 是一个切片操作，它会返回原数组的一个部分，而 `&` 符号则是获取这个部分的引用。所以 `&key[..32]` 的结果是 `&[u8]`，而不是 `&&[u8]`。

这是因为在Rust中，切片操作的结果是一个临时的切片，而临时的值不能有多个引用。所以，当你对 [`key[..32]`](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/process/text.rs") 使用 `&` 符号时，Rust会自动解引用 [`key`](vscode-file://vscode-app/Applications/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "src/process/text.rs")，使得你得到的是 `&[u8]` 而不是 `&&[u8]`。这是Rust的自动解引用（Deref）特性在起作用。