I am going to talk about some question in NumPy and try to explain why it works so well.

# Why is NumPy Faster?

## Operations

The operations in NumPy are written in **C and Fortran**, which are highly **optimized C operations** underneath. This avoids the overhead of Python's per-element loop and function calls. NumPy performs operations on **entire arrays at once** instead of element-wise loop, it's kinda like loop unrolling.
In python, each operation needs to do the type check, dynamic dispatch, memory allocation and reference counting stuff, while in NumPy we only need to do this kind of thing once, which reduces overheads.

## Data Layouts

In NumPy arrays they use **contiguous blocks of memory** rather than lists of array pointers in Python. Which means CPU can access elements in **sequential memory locations** and improves **cache efficiency**.
For example, the reshape and broadcasting in NumPy over `ndarray` is not allocating new memory space, but modifying the `stides`, `offset` and `shape`. So they are actually accessing the same piece of memory.

## Instruction Optimization

NumPy utilizes **vectorized CPU instructions (SIMD)** and **BLAS**, which means multiple elements can be accessed in parallel at hardware level.
