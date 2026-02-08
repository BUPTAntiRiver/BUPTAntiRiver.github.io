# Loop Optimization

## Loop Reordering

**Improve data locality**

Data movement(cache miss) is much more expensive, loop over row is much faster than loop over column if data is stored in row-major order.

## Loop Tiling

**Reduce cache miss**

To prevent the data size is too large to store. With tiling we can reduce the number of elements to access from $N\times N$ to $N \times \text{TILE\_SIZE}$ and even further to $\text{TILE\_SIZE}^2$

**Multilevel tiling**

## Loop Unrolling

**Reduce branching overheads**

# SIMD(single instruction, multiple data) programming

A parallel processing paradigm that applies a single instruction to multiple data elements simultaneously.

# Multithreading

Concurrent executing of multiple threads within a single process.

# CUDA programming
