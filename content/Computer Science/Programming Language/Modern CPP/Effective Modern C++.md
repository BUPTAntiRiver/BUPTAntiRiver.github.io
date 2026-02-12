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

`decltype` enables treating `auto` like `ParamType` rather than template type, so that it is more accurate for return types.

## Item 4: Know how to view deduced types.

In the book, the author introduces a library called boost, which produces accurate results, which implies results from compilers or IDEs may be wrong.

# Chapter 2 `auto`

## Item 5: `auto` may make mistakes but better that you!

## Item 6: Use the explicit typed initializer idiom when `auto` deduces undesired types.

Sometimes we still need explicit type declaration, because we may have proxy objects that improves performance but is not exactly the type we want.

For example, when indexing an element in `vector<bool>` we will get a proxy object due to `bool`s are single bit so they are compacted and accessed differently. So in this case if we use `auto x = bool_vec[y];` we will have `auto` deduced into some reference object but not a reference to the exact bool element.

So we may think we should go back to the old stuff using `bool x = bool_vec[y];` but to follow the `auto` idiom, _we should use `static_cast<bool>` on the right side and keep using `auto` on the left side_. Which makes everything clearer.

### Things to Remember

- "Invisible" proxy types can cause `auto` to deduce the "wrong" type for an initializing expression.
- The explicitly typed initializer idiom (using `static_cast<T>`) forces `auto` to deduce the type you want it to have.

# Chapter 3 Moving to Modern C++

In this chapter, we are going to explore the most famous and important features in C++11 and C++14: `auto`, smart pointers, move semantics, lambdas, concurrency.

## Item 7: Distinguish between `()` and `{}` when creating objects.

Two primary takeaways.

First, as a class author, you need to be aware of whether the overloads of constructors has arguments `std::initializer_list`, if it does, then client code with braced initialization will only see those.

Second, as a client, we must understand the difference between parentheses and braces when creating objects. For example with `std::vector` using parentheses like `std::vector<int> v1(10, 20);` will create 10 element vector with all elements 20, using braces like `std::vector<int> v2{10, 20};` will use `std::initializer_list` constructor to create a 2 element vector with value 10 and 20.

## Item 8: Prefer `nullptr` to `0` and `NULL`.

`nullptr` can be deduced into any forms of pointers, which is more flexible and easier to use than `0` and `NULL` in modern template cases.

Also, we should avoid overloading on integral and pointer types. Because in older version of C++ like C++98, the code usually use `0` and `NULL` which might work as `int` but not null pointer if we have integral or pointer overload.

## Item 9: Prefer alias declarations to `typedef`s.

For a super long type name like `std::unique_ptr<std::unordered_map<std::string, std::string>>`, we may want to give it a nickname.

```cpp
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS; // typedef
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>; // alias
```

`typedef` and alias seems to do the exactly same thing, so why we should prefer alias? Before get to that, it worth mentioning that alias declaration is easier to swallow, especially for case like function pointers:

```cpp
// FP is a synonym for a pointer to a function taking an int and
// a const std::string& and returning nothing
typedef void (*FP)(int, const std::string&); // typedef
// same meaning as above
using FP = void (*)(int, const std::string&); // alias declaration
```

However, using alias or `typedef` for function pointers is less common, so that is not a strong evidence.

The compelling reason is that alias declaration may be templatized easily, while `typedef` has to build such structure from scratch with the help of templatized `struct`.

```cpp
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>; // MyAllocList<T> is the synonym

MyAllocList<Widget> lw; // client code

template<typename T>
struct MyAllocList {
	typedef std::list<T, MyAlloc<T>> type; // MyAllocList<T>::type is the synonym
};

MyAllocList<Widget>::type lw; // client code
```

Also if we want to use `MyAllocList` as a member of a template class, we need to add `typename` before `typedef` synonym, because it is a _dependent type_ and that is a rule of C++ to add that keyword. In contrast, alias don't need this because they don't have the `::type`, which can make compilers wonder whether `type` could be a member of some unseen class.

## Item 10: Prefer scoped `enum` to unscoped `enum`.

Things to Remember

- C++98-style `enums` are now known as unscoped `enums`.
- Enumerators of scoped `enums` are visible only within the `enum`. They convert
  to other types only with a cast.
- Both scoped and unscoped `enums` support specification of the underlying type.
  The default underlying type for scoped `enums` is int. Unscoped `enums` have no
  default underlying type.
- Scoped `enums` may always be forward-declared. Unscoped `enums` may be
  forward-declared only if their declaration specifies an underlying type.

## Item 11: Prefer deleted functions to private undefined ones.

Things to Remember

- Prefer deleted functions to private undefined ones.
- Any function may be deleted, including non-member functions and template
  instantiations.

## Item 12: Declare overriding functions `override`.

Things to Remember

- Declare overriding functions override.
- Member function reference qualifiers make it possible to treat lvalue and
  rvalue objects (\*this) differently.

## Item 13: Prefer `const_iterator` to `iterator`.

Things to Remember

- Prefer `const_iterator`s to `iterator`s.
- In maximally generic code, prefer non-member versions of `begin`, `end`,
  `rbegin`, etc., over their member function counterparts.
