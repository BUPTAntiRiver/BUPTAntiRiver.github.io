# Struct

We can define struct in Rust like this:

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

And create an instance by stating the name and assigning values to attributes with key value pairs like this:

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

We can also make a struct data mutable, thus can change the value of its attributes.

We can create a new instance from other instance like this:

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

There is also tuple struct that does not specify attribute name, only has data type, so in order to use those values in the struct later, we have to destructure, which is naming those attributes.

You may wonder what is the ownership control of structs, in the example above we have `String` attribute, which is stored on heap, so it should be moved when transferring ownership, but if we have reference as a struct's attribute, the case will be more complex, and introduces lifetime, which shall be talked in the future.
