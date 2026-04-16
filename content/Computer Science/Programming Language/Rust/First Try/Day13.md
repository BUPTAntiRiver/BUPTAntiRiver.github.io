# Smart Pointers

In Rust, with the concept of ownership and borrowing, there is an additional difference between references and smart pointers: While references only borrow data, in many cases smart pointers _own_ the data they point to.

_Smart pointers_ are data structure that acts like a pointer but has additional information. Smart pointers are usually implemented using structs, and implement the `Deref` and `Drop` traits. The `Deref` trait allows an instance of the smart pointer to behave like a reference, so that you can write code that works with either references or smart pointers. The `Drop` trait allows you to customize the code that's run when an instance of the smart pointer goes out of the scope.

We’ll cover the most common smart pointers in the standard library:

- `Box<T>`, for allocating values on the heap
- `Rc<T>`, a reference counting type that enables multiple ownership
- `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time

## `Box<T>`

There is nothing special about `Box<T>` only that the data it points to is stored on heap, so if you want to have a type whose size cannot be known at compile time, you can use `Box<T>` points to it, because `Box<T>` itself has an exact size.

The practical usage is enabling _recursive types_ with `Box`. Suppose we have a type that can have another value of the same type as part of it self, it is called a recursive type and it poses an issue because Rust needs to know at compile time how much space a type takes up. But for recursive types, the space it takes could be infinite.

## `RefCell<T>`

Unlike `Rc<T>`, the `RefCell<T>` type represents single ownership over the data it holds. So, what makes `RefCell<T>` different from a type like `Box<T>`? Recall the borrowing rules you learned in Chapter 4:

- At any given time, you can have _either_ one mutable reference or any number of immutable references (but not both).
- References must always be valid.

With references and `Box<T>`, the borrowing rules’ invariants are enforced at compile time. With `RefCell<T>`, these invariants are enforced _at runtime_. With references, if you break these rules, you’ll get a compiler error. With `RefCell<T>`, if you break these rules, your program will panic and exit.

Here is a recap of the reasons to choose `Box<T>`, `Rc<T>`, or `RefCell<T>`:

- `Rc<T>` enables multiple owners of the same data; `Box<T>` and `RefCell<T>` have single owners.
- `Box<T>` allows immutable or mutable borrows checked at compile time; `Rc<T>` allows only immutable borrows checked at compile time; `RefCell<T>` allows immutable or mutable borrows checked at runtime.
- Because `RefCell<T>` allows mutable borrows checked at runtime, you can mutate the value inside the `RefCell<T>` even when the `RefCell<T>` is immutable.

## Memory Leak in Rust

With reference cycles, even Rust can have memory leak issue. We can see that Rust allows memory leaks by using `Rc<T>` and `RefCell<T>`: It’s possible to create references where items refer to each other in a cycle. This creates memory leaks because the reference count of each item in the cycle will never reach 0, and the values will never be dropped.

# Concurrency

The ultimate nightmare in programming. Let's see how to handle it with Rust.

Actually this part is very straight forward in the book. Just go to.

`Rc<T>` mentioned earlier is not safe across threads. We will need `Arc<T>` which is the atomic version.
