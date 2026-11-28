Today we will learn about the **Module System** in Rust. We have:

- **Packages**: a Cargo feature that lets you build, test and share crates
- **Crates**: a tree of modules that produces a library or executable
- **Modules and use**: let you control the organization, scope and privacy of paths
- **Paths**: a way of naming an item, such as a struct, function or module

# Packages and Crates

A _crate_ is the smallest amount of code that the Rust compiler considers at a time. If we run `rustc` rather than `cargo` and pass a single source code file, the compiler considers that file to be a crate. Crates can contain modules, and the modules may be defined in other files that get compiled with the crate.

A crate can have two forms: a binary crate and or a library crate. _Binary crates_ are programs that can be compiled into an executable that you can run. Each must have a function called `main` that defines what happens when the executable runs. All crates we created so far are binary crates.

_Library crates_ don't have a `main` function, and they don't compile to an executable. They are designed to be used multiple times across projects. Usually when we talk about "crates" we are saying library crates. Or we can just say "library".

A _package_ is a bundle of one or more crates. A package contains a `Cargo.toml` file that describes how to build these crates. A package can contain many binary crates but at most one library crate, also contains at least one crate.

So when we use `cargo new hello-world` to create a package, we will have a `Cargo.toml` file and has nothing in it. Also we have `src/main.rs` which is the _crate_ root of a binary crate with the same name of the package. We can create `src/lib.rs` to be the _crate_ root of _library crate_ with the same name of the package. _Crate root_ is the starting point of compile.

In the example above, we only have `src/main.rs` which means in this package we only have a binary crate named "hello-world".

# Control Scope and Privacy with Modules

The _module_ idea in Rust is similar to the class idea. When declaring modules, we use `mod module_name`, and compiler will search for the module's code under following places:

- Inline, which is the bracelets after `mod module_name` if we have
- In the file `src/module_name.rs`
- In the file `src/module_name/mod.rs`

Inside a module we can also define _sub-modules_, compiler will find code in these places:

- Inline
- In the file `src/module_name/sub_module_name.rs`
- In the file `src/module_name/sub_module_name/mod.rs`

To access code in the module, with `Foo` type as example, we can use `crate::module_name::sub_module_name::Foo`. The code within a module is _private_ from its parent modules by default. To make a module _public_, declare it with `pub mod`. If we only want to make some items in the module public, we can declare that item with `pub` before declarations.

Within a scope, we can use `use` as a short cut, we write `use crate::module_name::sub_module_name::Foo` and then next time we want to use `Foo` we can write `Foo` directly. Besides absolute path like this, we can also have relative path, if we are in `module_name` we can access `Foo` directly with `sub_module::Foo`. Also we can use `super` to access parent module.

`use` can be used on a path, not only specific type names. If we declare `use crate::module_name::sub_module_name` we can write code like `sub_module_name::Foo` directly in this scope.

## Use external packages

In the guess game demo we built, we used a package called `rand`. We add it by adding `rand = "0.8.5"` in `Cargo.toml` as a dependency, which tells Cargo to download the `rand` package and any dependencies from [crates.io](https://crates.io/) and make `rand` available to our project.

Then to bring `rand` definitions into the scope of our package, we have `use rand::Rng;`.

# Separating Modules into Different Files

It would be a good habit to split modules into multiple files so that your code looks neat.
