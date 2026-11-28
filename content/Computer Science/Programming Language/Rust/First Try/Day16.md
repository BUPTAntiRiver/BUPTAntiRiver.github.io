# Advanced Features

## Unsafe Rust

You can write `unsafe` code that Rust compiler cannot considered as safe code by adding the `unsafe` keyword. Also you can use Miri, the official Rust tool to dynamically detect undefined behavior during runtime.

Typical unsafe behaviors have:

- Dereference a raw pointer
- Call an unsafe function or method
- Access or modify a mutable static variable
- Implement an unsafe trait
- Access fields of a `unione`

## Advanced Traits

Just read. The documentation says these cases are rarely used, so just check it out. Actually I am too tired to write them down.

## Advanced Types

### Type Aliases

We can create type aliases just like `using` in C++. We can write code like `type Kilometers = i32;` and use it like `let x: Kilometer = 5;`. This is useful in reducing repetitive type name code.

### The Never Type That Never Returns

`continue` and `panic!` macro returns such kind of never type that never has a value. In arms of a `match` they won't be considered to have any type so they won't cause different type of some value in a `match` expression.

### Dynamically Sized Types

We have seen trait can have different types in it, which is just a dynamically shaped type. So we write code like `Box<dyn Trait>` need to have a `dyn`.

For generic functions, which needs to know the size of type at compile time, their restriction can be released by writing codes like:

```rust
fn generic<T: ?Sized>(t: &T) {
	// --snip--
}
```

## Advanced Functions and Closures

This part introduces the idea of function pointers and returning closures as return value. Which is very common if you have contact with languages like Python.

## Macros

We have used macros like `println!` throughout this journey in Rust. But what is a macro in essence?

### The Difference between Macros and Functions

Fundamentally, macros are a way of writing code that writes other code. It means that all of the macros *expand* to produce more code than the code you've written manually. So this makes me think of the head files in C. When we include a header we are actually copying everything into this file and then compile.

Function takes variables as input, and returns value as output, which has fixed number and types of parameters. Macros takes *code* as input, can accept variable number of arguments and generate new code before compilation.

There are two major methods to declare macros, the first and easy one is `macro_rules!`. It works like a `match` but matches on the stuff on the right hand side of it:

```rust
macro_rules! say_hello {
	() => {
		println!("Hello!");
	};
}

// use like this
say_hello!();
```

It matches the right hand side of it, in this case it matches the parentheses.

Another method is called procedure macros, which is more advanced and complicate. Not introduced here.