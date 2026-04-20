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
