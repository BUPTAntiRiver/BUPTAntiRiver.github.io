A **graphic processing unit (GPU)** is a specialized electronic circuit designed for digital image processing and to accelerate computer graphics. But nowadays it is widely used in boosting parallel computation tasks in machine learning. We all know that CPU is good at computing, so why does this GPU comes out? Let's dive into it.

# The difference between CPU and GPU

A common CPU is optimized to be as quick as possible to finish a task at a as low as possible latency, while keeping the ability to quickly switch between operations. It's nature is all about processing tasks in a serialized way.
A GPU is all about throughput optimization, allowing to push as many as possible tasks through is internals at once. It does so by being able to parallel process a task.
![[CPU and GPU 'cores'.png]]
This figure only shows the difference on number of cores, but that is not the only one.
The more important difference is that the number of memory caches, CPU has much more caches for each core, almost each core has multiple caches storing the instructions to be executed next. While GPU only has one cache for multiple cores. So this is why CPU is more capable of solving flexible tasks, and GPU handles repetitive tasks better.
**CPU architecture**
![[CPU architecture.png]]
**GPU architecture**
![[GPU architecture.png]]

# More About GPU

GPU uses SIMT architecture,which means _single instruction multiple threads_. Multiple threads make up a **thread block**, multiple thread blocks make up a **grid**. CPU invokes GPU at **grid** level.

Then the thread blocks are assigned to Streaming Multiprocessors (SM) to do the computation. In side the thread blocks, we have another group level, which is **warp**, usually consists of 32 threads, the threads in the same warp all perform the same operations at the same time on different data, it is kind like SIMD, which stands for single instruction multiple data, but the core design of CUDA is totally different from SIMD, and it is more flexible.

The total number of threads in each thread block is specified by the host code when a kernel is called. For a given grid of threads, the _number of threads in a block_ is available in a built-in variable named `blockDim`.

The `blockDim` variable is a struct with three unsigned integer fields ($x,y$ and $z$) that help user to organize the threads into a one-, two- or three-dimensional array. For one-dimensional array, we only use `blockDim.x` which indicates the total number of threads in each block, it is recommended that make each dimension of a thread block a multiple of 32 because each _warp_ has 32 threads.

## Declaration of Host/Device Function

| keyword      | call on | execute on | executed by                |
| ------------ | ------- | ---------- | -------------------------- |
| `__global__` | host    | device     | new grid of device threads |
| `__device__` | device  | device     | caller device thread       |
| `__host__`   | host    | host       | caller host thread         |

## Synchronization

In CUDA, each `syncthreads()` much be executed by all threads in a block, and are different synchronization points.

For example, `syncthreads()` in different branches in if-then-else statements are different so either all threads in the block should execute the then path or execute the else path. Or there will be undefined executions, like deadlock.
