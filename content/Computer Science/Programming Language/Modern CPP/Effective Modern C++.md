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
- Enumerators of scoped `enums` are visible only within the `enum`. They convert to other types only with a cast.
- Both scoped and unscoped `enums` support specification of the underlying type. The default underlying type for scoped `enums` is int. Unscoped `enums` have no default underlying type.
- Scoped `enums` may always be forward-declared. Unscoped `enums` may be forward-declared only if their declaration specifies an underlying type.

## Item 11: Prefer deleted functions to private undefined ones.

Things to Remember

- Prefer deleted functions to private undefined ones.
- Any function may be deleted, including non-member functions and template instantiations.

## Item 12: Declare overriding functions `override`.

Things to Remember

- Declare overriding functions override.
- Member function reference qualifiers make it possible to treat lvalue and rvalue objects (\*this) differently.

## Item 13: Prefer `const_iterator` to `iterator`.

Things to Remember

- Prefer `const_iterator`s to `iterator`s.
- In maximally generic code, prefer non-member versions of `begin`, `end`, `rbegin`, etc., over their member function counterparts.

## Item 14: Declare `noexcept` if functions won't emit exceptions.

Things to Remember

- `noexcept` is part of a function's interface, and that means that callers may depend on it.
- `noexcept` functions are more optimizable than non-`noexcept` functions.
- `noexcept` is particularly valuable for the move operations, `swap`, memory deallocation functions, and destructors.
- Most functions are exception-neutral rather than `noexcept`.

## Item 15: Use `constexpr` whenever possible.

Things to Remember

- `constexpr` objects are const and are initialized with values known during compilation.
- `constexpr` functions can produce compile-time results when called with arguments whose values are known during compilation.
- `constexpr` objects and functions may be used in a wider range of contexts than non-`constexpr` objects and functions.
- `constexpr` is part of an object's or function's interface.

## Item 16: Make `const` member functions thread safe.

Things to Remember

- Make `const` member functions thread safe unless you're _certain_ they'll never be used in a concurrent context. We still need to pay attention to `const` member function concurrent safety because there might be _mutable_ members.
- Use of `std::atomic` variables may offer better performance than a mutex, but they're suited for manipulation of only a single variable or memory location.

## Item 17: Understand special member function generation.

The two copy operations are independent: declaring one doesn't prevent compilers from generating the other, but the two move operations are not independent. If we declare either, that prevents compiler from generating the other.

Things to Remember

- The special member functions are those compilers may generate on their own: default constructor, destructor, copy operations, and move operations.
- Move operations are generated only for classes lacking explicitly declared move operations, copy operations, and a destructor.
- The copy constructor is generated only for classes lacking an explicitly declared copy constructor, and it's deleted if a move operation is declared. The copy assignment operator is generated only for classes lacking an explicitly declared copy assignment operator, and it's deleted if a move operation is declared. Generation of the copy operations in classes with an explicitly declared destructor is deprecated.
- Member function templates never suppress generation of special member functions.

# Chapter 4 Smart Pointers

Why a raw pointer is hard to love:

1. Its declaration won't tell us whether it points to an _object_ or an _array_.
2. Its declaration won't tell you whether you _should destroy_ what it points to or not.
3. Even though we know we should destroy what the pointer points to, there's _no way to know how_. Should you use a `delete` or some function to handle that.
4. Even though we know a `delete` is enough, we don't know whether to use single-object form `delete` or the array form `delete []`.
5. Even though we know how to delete and what to delete, we still cannot make sure we perform the destruction _exactly once_.
6. There is also no way to tell if the pointer dangles.

So we have _smart pointers_, they are wrappers around raw pointers that act much like the raw pointers they wrap, but that avoid many of their pitfalls.

There are different kinds of smart pointers like `std::unique_ptr`, `std::shared_ptr` and `std::weak_ptr`, they have different use and we should learn it.

## Item 18: Use `std::unique_ptr` for exclusive-ownership resource management.

`std::unique_ptr` is the closest to raw pointer, it performs exact the same for most of the instructions and is the same size as raw pointers. This means we can use them even in situations where memory and cycles are tight.

`std::unique_ptr` embodies _exclusive ownership_ semantics, just as its name. We can apply move, which transfers ownership from the source pointer to the destination pointer. Copying a `std::unique_ptr` is not allowed, it is a _move-only_ type.

**Things to Remember**

- `std::unique_ptr` is a small, fast, move-only smart pointer for managing resources with exclusive-ownership semantics.
- By default, resource destruction takes place via delete, but custom deleters can be specified. Stateful deleters and function pointers as deleters increase the size of `std::unique_ptr` objects.
- Converting a `std::unique_ptr` to a `std::shared_ptr` is easy.

## Item 19: Use `std::shared_ptr` for shared-ownership resource management.

Things to Remember

- `std::shared_ptrs` offer convenience approaching that of garbage collection for the shared lifetime management of arbitrary resources.
- Compared to `std::unique_ptr`, `std::shared_ptr` objects are typically twice as big, incur overhead for control blocks, and require atomic reference count manipulations.
- Default resource destruction is via delete, but custom deleters are supported. The type of the deleter has no effect on the type of the `std::shared_ptr`.
- Avoid creating `std::shared_ptrs` from variables of raw pointer type.

## Item 20: Use `weak_ptr` for `shared_ptr`-like pointers that can dangle.

It can be convenient to have a pointer like `std::shared_ptr` that doesn't affect an object's reference count.

If you check the weak pointer API, you will wonder how could a `std::weak_ptr` could be useful. Because it can't be dereferenced nor can they be tested for nullness. This is due to `std::weak_ptr` isn't a standalone smart pointer, it's an augmentation to `std::shared_ptr`. It is created from `std::shared_ptr`.

We can use `std::weak_ptr<Widget> wpw(spw);` to create a weak pointer that points to the same object as the shared pointer. Then we can use `wpw.expired()` to check whether the pointer dangles. Why we can't dereference or test nullness of weak pointer is because the test and use _may have concurrency problem_, like not null for test but then being destroyed by the shared pointer and still try to use the weak pointer will have problem.

Instead, we have an atomic operation: `wpw.lock()` which will test nullness and return a shared pointer to the object if not null else `nullptr`. Or you can use `std::weak_ptr` to construct a new `std::shared_ptr`, if expired, it will throw `std::bad_weak_ptr`.

Things to Remember

- Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle.
- Potential use cases for `std::weak_ptr` include caching, observer lists, and the
  prevention of `std::shared_ptr` cycles.

## Item 21: Prefer `std::make_unique` and `std::make_shared` to direct use of `new`.

Creation with `make` function and `new` is like:

```cpp
auto upw1(std::make_unique<Widget>());
std::unique_ptr<Widget> upw2(new Widget);

auto spw1(std::make_shared<Widget>());
std::shared_ptr<Widget> spw2(new Widget);
```

With `make` function, we can reduce code duplication.

Things to Remember

- Compared to direct use of new, make functions eliminate source code duplication, improve exception safety, and, for `std::make_shared` and `std::allocate_shared`, generate code that's smaller and faster.
- Situations where use of make functions is inappropriate include the need to specify custom deleters and a desire to pass braced initializers.
- For `std::shared_ptr`s, additional situations where make functions may be ill-advised include (1) classes with custom memory management and (2) systems with memory concerns, very large objects, and `std::weak_ptr`s that outlive the corresponding `std::shared_ptr`s.

## Item 22: When using the Pimpl Idiom, define special member functions in the implementation file.

Pimpl means pointer to implementation.

Things to Remember

- The Pimpl Idiom decreases build times by reducing compilation dependencies between class clients and class implementations.
- For `std::unique_ptr` pImpl pointers, declare special member functions in the class header, but implement them in the implementation file. Do this even if the default function implementations are acceptable.
- The above advice applies to `std::unique_ptr`, but not to `std::shared_ptr`.

# Chapter 5 Rvalue references, Move Semantics, and Perfect Forwarding

It is important to bear in mind in this chapter that a parameter is always a lvalue, even if its type is a rvalue reference.

## Item 23: Understand `std::move` and `std::forward`.

Things to Remember

- `std::move` performs an unconditional cast to an rvalue. In and of itself, it doesn't move anything.
- `std::forward` casts its argument to an rvalue only if that argument is bound to an rvalue.
- Neither `std::move` nor `std::forward` do anything at runtime.

## Item 24: Distinguish universal references from rvalue references.

Things to Remember

- If a function template parameter has type `T&&` for a deduced type `T`, or if an object is declared using `auto&&`, the parameter or object is a universal reference.
- If the form of the type declaration isn't precisely `type&&`, or if type deduction does not occur, `type&&` denotes an rvalue reference.
- Universal references correspond to rvalue references if they're initialized with rvalues. They correspond to lvalue references if they're initialized with lvalues.

## Item 25: Use `std::move` for rvalue references, `std::forward` for universal references.

`std::move` is used when you know that you own this object and you are giving it to other owners, `std::forward` is used when you received the object from other owner and you don't know it should be a rvalue or lvalue, so you treat it more generally, which is just forward it to other owners.

For locally created objects, you don't need to do `move` or `forward`, the compiler is smart enough to handle and optimize that, just leave it alone.

Things to Remember

- Apply `std::move` to rvalue references and `std::forward` to universal references the last time each is used.
- Do the same thing for rvalue references and universal references being returned from functions that return by value.
- Never apply `std::move` or `std::forward` to local objects if they would otherwise be eligible for the return value optimization.

## Item 26: Avoid overloading on universal references.

Since universal reference is _universal_ if we use it in overloading, it will match more cases than you think.

Things to Remember

- Overloading on universal references almost always leads to the universal reference overload being called more frequently than expected.
- Perfect-forwarding constructors are especially problematic, because they're typically better matches than copy constructors for non-`const` lvalues, and they can hijack derived class calls to base class copy and move constructors.
