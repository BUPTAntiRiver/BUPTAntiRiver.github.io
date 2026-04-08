This time we talk about collections in Rust. We will introduce _vector, String and hash map_.

# Vector

To declare a vector in Rust, we do:

```rust
let v: Vec<i8> = Vec::new();
```

To push new value to vector, we first need to make the vector mutable. Also elements of immutable vector can not be modified, so when we iterate over a vector, we need to check this first.

# String

`String` is different from `&str` string slice type, which is the only one string type in the core language. `String` is provided by Rust's standard library rather that coded into the core.

The interesting point of Rust string is that it computes `String` length according to the bytes used in the utf-8 encoding of the string. So we are not allowed to index `String` with integer, and when sliced in the middle of a multiple bytes character, Rust will panic.

So to iterate over the individual Unicode scalar values of `String`, we will need the `chars` method. Or you can use `bytes` method to iterate over the bytes.

# Hash Map

Nothing special, but be careful with the ownership.

We insert with `map.insert(key, value)` and access with `map.entry(key)`. We use `or_insert` to handle cases when key is not available.
