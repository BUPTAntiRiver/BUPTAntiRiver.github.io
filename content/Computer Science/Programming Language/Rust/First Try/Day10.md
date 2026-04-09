# Error Handling

Rust groups errors into two groups: recoverable and unrecoverable. Recoverable errors might be File Not Found, and unrecoverable error might be reading dirty data and writing unwillingly.

In rust we have distinct methods to handle different kinds of errors. For recoverable errors, we have type `Result<T, E>` and use `panic!` macro to stop execution when encountering unrecoverable error.

## Unrecoverable Errors

There are two ways to cause panic: running some bad code that cause panic or explicitly calling the `panic!` macro.

When panics happen, they will print a failure message, unwind and clean up the stack and quit. So actually, panics do more than simply abort, and if you want it to behave just like simple abort to quit faster, you can add `panic = 'abort'` to the appropriate `profile` section in `Cargo.toml`.

## Recoverable Errors

The `Result` enum is defined to have two variants:

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

Since then we can have different behaviors on success or failure base on `Ok` or `Err`. Use file open as example:

```rust
use std::fs::File;

fn main() {
	let greeting_file_result = File::open("hello.txt");
}
```

In this case, `File::open` will return `Result`, type `T` will depend on the implementation of `File::open` and type `E`, the error value will be `std::io::Error`. Then we can use `match` to handle the `Result`. You can also use `match` on the `Err` for further operation. Like create the file if not exist. But that would seem so complicated and cumbersome, so remember the `expect` or `unwrap`? They are shortcuts for `Result` handling.

### Propagating Errors

Since error information are stored in `Result` type, we can pass it as return value so that enable the caller to handle it.

We also have a `?` shortcut for this.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

The function decorated with `?` will return `Ok` wrapping return value if it works and return the _whole_ function `Err` if something wrong happens. And `?` can only be used when the function `?` lives in has compatible return type with `?`. Also it can be used on `Option` case, it will return `None` early rather than `Err`.

## When to panic?

How do we decide when we should `panic` or we should just return `Result`? When we call `panic` it is not recoverable, you give caller no chance to handle the situation. If you return `Result` then you make the decision that a situation is recoverable on behalf of the caller. So return `Result` is a more flexible default choice when the code might fail.
