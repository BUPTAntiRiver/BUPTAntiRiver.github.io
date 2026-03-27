Today we start reading the book: [The Rust Programming Language](https://doc.rust-lang.org/book/title-page.html).

I made a simple number guessing game by following the tutorial and find some interesting stuff in rust.

The condition judge in Rust is very interesting, we have `match` and an `enum` called `std::cmp::Ordering` to achieve smaller, equal or greater comparing. It works really similar to `switch` and `case`.

Also rust seems to have dynamic type? We just use `let x = 5;` to bind a variable, well actually it is not dynamic type, we cannot assign a `string` type to `x` with such initialization. Currently `x` is _immutable_, which means we cannot assign another value to it, if we want it to be _mutable_, we should declare it like `let mut x = 5;`.

**Constants** in rust should be name with all uppercase with underscores between words like:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

**Shadowing**. In rust you _can_ declare a new variable with the same name as a previous variable (oh! we are writing python). We say that the first variable is _shadowed_ by the second. Shadow is creating a new variable, so the data type can be different, but mutable cannot do that.

**Tuple.** The tuple in rust looks like this

```rust
let x: (i32, f64, u8) = (500, 6.4, 1);
```

and can be accessed with index like `x.0`. Tuple can have different data type elements.

**Array.** Every element in array must have the same type. Also it has a fixed length. Arrays are useful when you want your data to be allocated on the _stack_. Its usage is just like C++, but it has index out of range detection, so that prevents invalid memory access.

**Function.** If we end a line with `;` then it becomes a statement, which has no return value, so you cannot assign a statement to a variable. But if we don't have a `;`, it becomes a expression, which evaluate to a resultant value.

So in function, if we want it to return something, then in the last line, we just write the return expression with no `;` at the end.

```rust
fn plus_one(x: i32) -> i32 {
	x + 1
}
```
