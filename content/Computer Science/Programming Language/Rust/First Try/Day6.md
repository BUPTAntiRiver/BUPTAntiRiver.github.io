It's been a long time since last time play with Rust, internship quite exhausted me.

Now we enter the chapter of:

# Enums and Pattern Matching

Remember the awful non control flow style comparison code we wrote in the guess number demo? Maybe this chapter will explain why we do it like that.

What is `enum`? It tells you what possible values can be in this set.

```rust
enum IpAddrKind {
	V4,
	V6,
}
```

Without mentioning anymore info, it acts like a category.

We can also assign specific values to `enum` variants (possible value of `enum`):

```rust
enum IpAddr {
	V4(String),
	V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
```

The variants in an `enum` can have different types or no type. And just like `struct`, `enum` variants can also have methods with `impl`.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
	fn call(&self) {
		// method body would be defined here
	}
}

let m = Message::Write(String::from("hello"));
// the self when m calls will have the value Message::Write(String::from("hello"))
m.call();
```

So `enum` actually is very similar to parent class in my opinion, because we can have same methods for many variants under the `enum`, but it is also very different, because variants can have nothing in common and they don't inherit anything from `enum`, `enum` it self has no attributes, it is just a set.

But how to write the body of `call`? We still need to figure this out.

## The `Option` Enum

This section explores a case study of `Option`, which is another enum defined by the standard library. The `Option` type means a value can be something or nothing, just like `Optional` in C++.

In Rust, we don't have `null` but use `Option<T>` to express null, the enum can encode the concept of a value being present or absent.

```rust
enum Option<T> {
	None,
	Some(T),
}
```

This enum is so widely used, so it is included in the prelude, we don't need to bring it to scope explicitly. The `<T>` syntax is called a generic type parameter, means that the `Some` variant of the `Option` enum can hold one piece of data of _any_ type.

```rust
let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;
```

The type of `some_number` is `Option<i32>` and the type of `some_char` is `Option<char>`, which is a different type.

When we have a `Some` value, we know that the value is present, and the value is held within `Some`. When we have a `None` value, we know it is absent, the same thing as null. The benefit of not having null but `Option<T>` is `Option<T>` and `T` are _different_ types, the compiler won't let us use an `Option<T>` as if it were definitely a valid value.

For example, this code won't compile:

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

We will have error code like:

```sh
error[E0277]: cannot add `Option<i8>` to `i8`
   --> src/main.rs:5:17
    |
  5 |     let sum = x + y;
    |                 ^ no implementation for `i8 + Option<i8>`
    |
    = help: the trait `Add<Option<i8>>` is not implemented for `i8`
help: the following other types implement trait `Add<Rhs>`
   --> /home/xiaoqianhe/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ops/arith.rs:99:9
    |
 99 |         impl const Add for $t {
    |         ^^^^^^^^^^^^^^^^^^^^^ `i8` implements `Add`
...
114 | add_impl! { usize u8 u16 u32 u64 u128 isize i8 i16 i32 i64 i128 f16 f32 f64 f128 }
    | ---------------------------------------------------------------------------------- in this macro invocation
    |
   ::: /home/xiaoqianhe/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/internal_macros.rs:22:9
    |
 22 |         impl const $imp<$u> for &$t {
    |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^ `&i8` implements `Add<i8>`
...
 33 |         impl const $imp<&$u> for $t {
    |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^ `i8` implements `Add<&i8>`
...
 44 |         impl const $imp<&$u> for &$t {
    |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `&i8` implements `Add`
    = note: this error originates in the macro `add_impl` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
```

So we have to convert `Option<T>` to `T` before you can perform `T` operations. There are a lot of conversion methods, you may check the [document](https://doc.rust-lang.org/std/option/enum.Option.html). The book says we should be familiar with that, it helps a lot.

To use `Option<T>` well, we need to handle each variant, like the inner of `Some` is valid, or case that we are dealing with a `None`. The `match` expression is a control flow construct that does just this when used with enums.
