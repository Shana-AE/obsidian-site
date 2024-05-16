```rust
pub trait AsRef<T: ?Sized> {

/// Converts this type into a shared reference of the (usually inferred) input type.

#[stable(feature = "rust1", since = "1.0.0")]

fn as_ref(&self) -> &T;

}
```

`?Sized` 表示参数可能是固定大小的类型，也可能是未定大小的类型，是rust中的唯一宽松约束

用于将一个类型转换为另一个类型的引用

这个trait通常用于编写可以接受多种类型参数的函数。例如，如果你有一个函数需要一个字符串切片，你可以让这个函数接受实现了`AsRef<str>`的任何类型，这样就可以接受`String`或`str`类型的参数。