# Generic Types, Traits and Lifetimes

If we have tried other programming languages, we will know a bit about Generic Types, such as template in C++ (it is said to have some difference). Generic Types help us to reduce code duplication.

And in Rust, if we have a function that can find the largest element in a list, and we want to make it workable on any list, we can write it like:

```rust
fn largest<T>(list: &[T]) -> &T {
	let mut largest = &list[0];

	for item in list {
		if item > largest {
			largest = item;
		}
	}

	largest
}
```

But if we try it we will find out the syntax checker says "binary operation '>' cannot be applied to type '&T'". And see "consider restricting type parameter `T` with trait `PartialOrd` `std::cmp::PartialOrd`". What is trait? Currently, I think it is something that tells what "traits" does the generic type have, just like it says, a "restriction".

Generic types can be used in functions, enum, methods and you can even create specific methods on some generic data type.

People may question, generic type is so convenient, is there any runtime cost? The answer is no. Because in Rust we will only create corresponding generic type function base on the information inferred at compile time. So it will only have minimal code generated.

# Defining Shared Behavior with Traits

Traits in Rust define **shared behavior** across different types, similar to interfaces in other languages.

- A **trait** declares method signatures (a contract) that types can implement.
- Types implement a trait using `impl Trait for Type`, providing concrete behavior.
- Traits allow writing **generic functions** that operate on any type implementing certain behavior (via trait bounds).
- Traits can include **default method implementations**, which types can use or override.
- To use trait methods, the trait must be **in scope**.
- Rust enforces the **orphan rule**: you can implement a trait for a type only if either the trait or the type is defined in your crate.

**In short:** traits enable abstraction and code reuse by defining shared behavior that multiple types can implement safely and consistently.

# Lifetimes

We mentioned lifetime when we talked about reference, which is the scope for which that reference is valid. Most of the time, lifetime is implicit and inferred, just like types. But when multiple types can be possible we must annotate type explicitly, so does lifetime.

For example if we have code like this:

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {result}");
}

fn longest(x: &str, y: &str) -> &str {
	if x.len() > y.len() { x } else { y }
}
```

It won't compile. Because we don't know result will be reference to `x` or `y` so its lifetime has two possibilities, and compiler won't know if it works well, since then cannot ensure safety.

In this case, we need lifetime annotations;

```rust
&i32        // a reference
&'a i32     // a reference with explicit lifetime
&'a mut i32 // a mutable reference with explicit lifetime
```

If we turn our code into this, it will compile and work:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
	if x.len() > y.len() { x } else { y }
}
```

It means result will have same lifetime as input `x y`.

If we use lifetime generic annotation in struct like this:

```rust
struct ImportantExcerpt<'a> {
	part: &'a str,
}
```

This means the lifetime of the whole struct should be the same as its `part` when created. There are some other rules about lifetime, and compiler has been improved to avoid user writing too much tedious `'a`, you may check the document for further exploration.

**Static** lifetime means that it will live for the entire duration of the program.

# Summary

Let's combine everything together:

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {ann}");
    if x.len() > y.len() { x } else { y }
}
```

It will look like this. For further more complex case study, read [Rust Reference](https://doc.rust-lang.org/reference/trait-bounds.html). But I think if you really meet such complex scenario, it might be someone has not that satisfying coding ability and we should try to improve it.
