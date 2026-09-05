# Control Flow

Keep reading **the book**. So actually in rust we do have simple and basic control flow keywords like `if`, `loop`, `while` and `for`. So why in the guess game we have to use a redundant enum? Maybe just to terrify you. So let's try to rewrite the guess game with the common way first.

# Ownership

It says that ownership is the most unique feature of rust, let's see what is its difference comparing ownership in C++.

So it is said that there are different methods to manage memory: like garbage collector that regularly looks for no-longer-used memory as the program runs; or like in C, we explicitly allocate and free the memory. Rust decides to solve such problem in compile time, **if the ownership system rules is violated**, the program **won't compile**.

**Ownership Rules:**

- Each value in rust has a _owner_.
- There can only be one owner at a time.
- When the owner goes out of the scope, the value will be dropped.

Scope is easy to understand, they are _bracelets_, but what is a owner?

To illustrate the role of owner, we need a data type that is more complex than those basic data types we have seen. Those basic types are typically of a known size and can be stored on stack and popped out simply when leaving their scope. So we want to see some data that is stored on the heap, and explore how does rust handle it, here we take `String` type as example.

When a variable exits its scope, rust will call a function called `drop` to free the memory. Just like RAII that controls item's lifetime.

So for such complex data types that stored on **_heap_**, rust will apply **_move_** automatically for you. Which avoids double free and saves resource. Which means if we write code like:

```rust
let s1 = String::from("Hello");
let s2 = s1;
```

`s1` value will be moved to `s2`, and we can only use `s2` now. If you really want to do copy, which can be expensive, you can do `s1.clone()`.

For variables on **_stack_**, rust will do copy. And when we pass variables as parameters to functions, they work just like value assignment, heap variables will be moved and stack variables will be copied. And move is another name of passing ownership. (copy is also ownership transfer)

Well, not every time we do assignment means we want to transfer ownership, for such requirement, rust has a feature called: **_reference_**.

# Reference

A reference is like a pointer in that it’s an address we can follow to access the data stored at that address; that data is owned by some other variable. Unlike a pointer, a reference is guaranteed to point to a valid value of a particular type for the life of that reference.

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{s1}' is {len}.");
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

So in this case, we can keep ownership of `s1`, and use it later. But we cannot change `s` in `calculate_length` here, because inside this function, we only
borrow the value with a reference, and never has ownership. To enable modification, we should do some little tricks, which is change the reference to a mutable reference like this:

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

But with mutable reference, there is a restriction, you are _not allowed to create two mutable reference to same data at the same time_. This is quite reasonable, in order to prevent _data race_, which means multiple access to same data at same time, kinda like classic multi-thread problem.

Also we cannot have mutable reference while we have an immutable one in scope. This is just like reader and writer lock! But rust prevents such kind of problem at compile level.

The slice in rust is a reference to part of the data, so previous rules ensure our slice will be valid at compile level. By the way, the syntax of rust slice is `[..]` which is slice in Python `[:]` lies down.
