Resource: [book](https://ananyapam7.github.io/resources/C++/Scott_Meyers_Effective_Modern_C++.pdf)

# Chapter 1 Deducing Types

## Item 1: Understand template type deduction.

For a function template looks like:

```cpp
template<typename T>
void f(ParamType param);
```

A call can look like:

```cpp
f(expr); // call f with some expression
```

During compilation, the compilers use `expr` to deduce two types: one for `T` and one for `ParamType`.

During template type deduction, arguments that are references are treated as non-references, like `int&` will be deduced into `int` for template type, but `ParamType` will remember the decorations like `&` or `const`.

When deducing types for universal reference parameters, lvalue arguments get special treatments. Their template type also deduces decorations like `ParamType`. For rvalue everything stays the same.

When deducing types for by-value parameters, we are actually create a copy with the same value as the arguments, so the `const` or `volatile` are treated as non-`const` and non-`volatile`.

Also during template type deduction, arguments that are array or function names decay to pointers, unless they're used to initialize references.

## Item 2: Understand `auto` type deduction.

`auto` type deduction is almost identical to template type deduction, because in practice they are just very similar. For example, we may have code like:

```cpp
auto x = 27;
const auto cx = x;
const auto& rx = x;
```

`auto` is just like the `T` in template type, and `ParamType` is the total type specifier.

In item 1 we have 3 cases for template type deduction, so does `auto`, they act almost but with 1 exception as I have mentioned.

The special case comes from different initialization methods we have, use `int` for example:

```cpp
auto x1 = 27;
auto x2(27);
auto x3 = {27};
auto x4{27};
```

If we use `int` instead of `auto` this would all work, and give you a variable of type `int` and value 27. But for `auto` in the two latter case, the deduced type is a `std::initializer_list<int>` containing a single element with value 27!

## Item 3: Understand `decltype`.
