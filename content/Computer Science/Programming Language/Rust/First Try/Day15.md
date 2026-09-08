# Patterns and Matching

What is pattern in Rust? It is some sort of data shape, if the data matches the pattern, means it has specific data shape and corresponding code can be operated on it. So it can be used in the `match` syntax we mentioned before and `if let` expression.

## All the Places Patterns Can Be Used

### `match` Arms

```rust
match VALUE {
	PATTERN => EXPRESSION,
	PATTERN => EXPRESSION,
	PATTERN => EXPRESSION,
}
```

The particular pattern `_` will match anything, but it never binds to a variable, so it's often used in the last match pattern.

### Conditional `if let` Expressions

`if let` expressions is actually a shorter way to write the equivalent of a `match` that only matches one case. `if let` can be mixed with other flow control keywords like `else` if nothing matches or `else if` to check another boolean variable or `else if let` to do another match.

### `while let` Conditional Loops

Since `let` can be used in `if` condition, it can be also used in `while` condition intuitively. It allows a `while` loop to run for as long as a pattern continues to match.

### `for` Loops

Patterns are not only enum, tuple like data can also be pattern. The value that directly after the keyword `for` is a pattern. For example, in `for x in y` the `x` is a pattern, in `for (index, value) in vec.iter().enumerate()` the `(index, value)` is a pattern. We can use a pattern to destruct a tuple.

### `let` and Function Parameter

In `let x = 5;` the `x` here is also a pattern, also in function parameters. I think what we are doing in the function scope is actually calling something like `let param = <passed_in_value>` inherently.

## Some Pattern will Never Fail to Match

In function parameters the pattern match shall not fail, because if it fails it means we break the rules of that function and it becomes meaningless.

Other cases are code like `let x = 5;` we cannot handle the failed case like `let x = 5 else {};`, this code won't compile because it will never fail so we have useless code here.

While for code like `let Some(x) = some_option_value else {};` the lateral `else` could not be omitted, because we must handle the `None` case.

## Pattern Syntax

This part just list all kinds of usage of `match`. Just read.
