---
tags:
  - TORESOLVE
  - rust
---

```rust
let json_record = headers.iter().zip(record.iter()).collect::<Value>();
drop(record);

println!("json_record {:?}", json_record);
```
iter() 过程中是不可变引用，那collect过去之后是copy还是clone了吗？为什么后续record被drop之后json_record依然可用