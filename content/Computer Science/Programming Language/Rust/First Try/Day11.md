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

But if we try it we will find out the syntax checker says "binary operation '>' cannot be applied to type '&T'". And see "consider restricting type parameter `T` with trait `PartialOrd` `std::cmp::PartialOrd`". What is trait? 
