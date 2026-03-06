# Idea

Flash attention is a family of algorithms to compute the attention **faster** with **less memory** without changing the outputs. Nowadays we need to deal with longer and longer sequences, so normal attention isn't enough.

# Problem in normal attention

$$
\text{Attention}(Q,K,V)=\text{softmax}\left(  \frac{QK^T}{\sqrt{ d }} \right)V
$$

Suppose sequence length $n$ and hidden size $d$, then $QK^T$ would be of size $n \times n$. That would take $O(n^{2})$ memory and $O(n^{2}d)$ for computation. So for **long** sequence, this would be very **expensive**.

# FlashAttention

[Paper Link](http://arxiv.org/abs/2205.14135)

## Introduction

GPU has **three memory hierarchy**, from highest bandwidth to lowest are: GPU SRAM (20 MB), GPU HBM (40 GB) and MAIN MEMORY (>1 TB). This is the **IO cost** we should consider in real scene, which is the **bottleneck** of most operations in Transformer.

We want to keep data in SRAM to achieve higher average bandwidth, but SRAM also has smallest size, so it won't be possible to store the whole K, V matrix in it. Instead, we choose to loop through blocks of **K**, **V** matrices.

Flash attention enables **faster training** and **higher quality models (longer sequence length to hold, faster is better!)**.

## Background

### Kernel Fusion

When there are multiple operations applied to the **same input**, the input can be loaded **once** from HBM, and the Compilers can automatically fuse many element-wise operations. However in the context of model training, the intermediate still need to be written to HBM to save for the backward process, so the **naive kernel fusion won't work very well**.

## FlashAttention itself

We want to compute with **fewer HBM IO** and **without storing large intermediate** matrices for the backward pass. I will show the implementation detail of FlashAttention-2 later.

### Tiling and Recomputation

_Tiling_

Split the input $Q,K,V$ into blocks and load them from slow HBM to fast SRAM. In order to ensure softmax stability, we need to compute $m(x),l(x)$ for all the blocks to do so, this also avoids softmax over the whole large matrix which is very slow.

_Recomputation_

$S=QK^T$ and $P=\text{softmax}\left(  \frac{S}{\sqrt{ d }}  \right)$ are needed to compute gradients of $Q,K,V$, but we can recompute them with output $O$ and softmax normalization statistics $m,l$ in SRAM.

### Why is FA better?

For naive attention, when computing the score between $Q,K$, the hardware may just acts like tiling, but after it finished computation of 1 block of $Q$ and whole sequence of $K$, it will store the result and do the next $Q$, rather than what we are doing in the FA, which is keep computing local max and sum and doing updates, then compute output with loaded $V$. These are three separates steps in naive attention, so it will have at least 3 times of FA HBM access.

### Block-Sparse FlashAttention

This implementation is a **approximate** attention. It has one predefined block sparsity matrix $M$, to mask out zero blocks.

# FlashAttention-2

[Paper Link](http://arxiv.org/abs/2307.08691)

## Introduction

Original FlashAttention has suboptimal work partitioning between different thread blocks and warps on GPU, causing either **low-occupancy** and **unnecessary shared memory reads/writes**. And here FlashAttention-2 has better **parallelism** and **work partitioning**.

## Background

See above. [[#Introduction]]

## FlashAttention-2 itself

### Algorithm

To reduce NON-matmul operations, we can skip rescale of output after each step, but only **rescale the final** output $\hat{O}^{(\text{last})}$ by $(l^{\text{(last)}})^{-1}$ to get the right output.

We also don't have to save both $m$ and $l$ to recompute for backward, instead we only need to save **logsumexp** $L=m+\log(l)$.

### Parallelization

Additionally parallelize over sequence length dimension.

### Partitioning

In forward pass, originally we split over $K,V$, now we split $Q$ across warps, while keeping $K,V$ accessible to all warps, which reduces communication between warps and shared memory IO.

In backward pass, we also try to **avoid K-split**, but still requires some synchronization.

Block size can also be tuned to achieve better performance.

## Implementation

### Forward Pass

In forward pass, what we have are the input matrices $\mathbf{Q,K,V}\in \mathbb{R}^{N\times d}$ in HBM, and suppose block sizes are $B_{c},B_{r}$. Usually we take $\mathbf{Q}$ as rows, and $\mathbf{K,V}$ as cols.

So now, we divide $\mathbf{Q,K,V}$ into corresponding blocks, and prepare the output space $\mathbf{O,L}$ for output and logsumexp (used in backward recompute). $\mathbf{O}$ has shape $\mathbb{R}^{N\times d}$ and is divided just like $\mathbf{Q}$, $\mathbf{L}$ has shape $\mathbb{R}^{N}$ so its tile shape is $B_{r}$.

Then, for _each block_ of $\mathbf{Q}$, which is $\mathbf{Q}_{i}$, we load it from HBM to on-chip SRAM. After that, we initialize $\mathbf{O}_{i}^{(0)}=(0)_{B_{r}\times d}\in \mathbb{R}^{B_{r}\times d}, l_{i}^{(0)}=(0)_{B_{r}}\in \mathbb{R}^{B_{r}},m_{i}^{(0)}=(-\infty)_{B_{r}}\in \mathbb{R}^{B_{r}}$. The upper index stands for block indices of $\mathbf{K,V}$, and the lower one is $\mathbf{Q}$ row block index. $\mathbf{O}$ for output, $l$ for sum, $m$ for max.

**Within the loop** of $\mathbf{Q}$, for _each block_ of $\mathbf{K,V}$, which is $\mathbf{K}_{j},\mathbf{V}_{j}$, we load them from HBM to chip. Compute the score $\mathbf{S}^{(j)}_{i}=\mathbf{Q}_{i}\mathbf{K}_{j}^\top$, and update global max $m_{i}^{(j)}=\max(m_{i}^{(j-1)},\text{rowsum}(\mathbf{S}_{i}^{(j)}))$, compute stable probability $\tilde{\mathbf{P}}_{i}^{(j)}=\exp(\mathbf{S}_{i}^{(j)}-m_{i}^{(j)})$.

And also the running sum: $l_{i}^{(j)}=e^{m_{i}^{(j-1)}-m_{i}^{(j)}}l_{i}^{(j-1)}+\text{rowsum}(\tilde{\mathbf{P}}_{i}^{(j)})$, we scale the old sum with newest max value and add the new probabilities to it.

Finally, on-chip, we compute the output $\mathbf{O}_{i}^{(j)}=\text{diag}(e^{m_{i}^{(j)}-m_{i}^{(j-1)}})^{-1}\mathbf{O}_{i}^{(j-1)}+\tilde{\mathbf{P}}_{i}^{(j)}\mathbf{V}_{j}$. Currently, it is not normalized, only after running through all the keys and values we can do the normalization.

After finishing the _for loop over keys and values_, we compute $\mathbf{O}_{i}=\text{diag}(l_{i}^{(T_{c})})^{-1}\mathbf{O}_{i}^{(T_{c})}$, and logsumexp $L_{i}=m_{i}^{(T_{c})}+\log(l_{i}^{(T_{c})})$, and write them to HBM.

### Backward Pass

In backward pass, we have the inputs $\mathbf{Q,K,V}$ as well as output $\mathbf{O}$ and top down gradient $\mathbf{dO}$, they all have shape $\mathbb{R}^{N\times d}$ and live on HBM. Also we have logsumexp $L\in \mathbb{R}^{N}$ in HBM.

This part is more straight forward in the paper. Comparing to the original paper, the main difference is in the way that it calculates, because we are storing only logsumexp in this version.

# FlashAttention-3

[Paper Link](http://arxiv.org/abs/2407.08608)

## Introduction

This method is specially optimized for Hopper architecture GPUs by NVIDIA. In this paper, the researchers used NVIDIA Hopper H100 SXM5 GPU as benchmark, its Thread-Memory Hierarchy looks like this:

| Hardware Level     | Parallel Agent                              | Data Locale     |
| ------------------ | ------------------------------------------- | --------------- |
| Chip               | Grid                                        | GMEM            |
| GPC                | Threadblock Clusters                        | L2              |
| SM (Shared Memory) | Threadblock (CTA cooperative thread arrays) | SMEM            |
| Thread             | Thread                                      | RMEM (Register) |

From top to bottom the capacity grows smaller but bandwidth becomes higher.

## Algorithm

### Producer-Consumer

Producer for data loading and consumer for computation. WGMMA for matmul and TMA for loads/stores.

And in computation, we can further improve scheduling with **pingpong scheduling**.

Matmul throughput is much bigger than non-matmul operations. Since exponential is performed on a separate hardware unit, we want to schedule it when Tensor Core is performing matmul. When one warpgroup is doing the next matmul, the other warpgroup can do softmax. So that **no unit is idle**.

### Intra-warpgroup overlapping

Even with one consumer warpgroup we can overlap some operations in the softmax with some instructions in the GEMMs. There all 3 parts of operations, matmul, softmax and matmul again, so when computing the attention, we can compute matmul and softmax of block 0 first, then compute the first matmul of block 1 rather than second matmul of block 0, because we decide to compute it when the other unit is computing softmax of block 1, which achieves overlapping. The figure in the paper shows clearly.

#### Some practical problems

Compiler reordering, high register pressure, and 3 or more stage pipelining which has more overlaps.

### Low precision with FP8

#### Efficiency

The $Q,K,V$ are typically given as contiguous in the head dimension, while to satisfy the k-major constraint, we need $V$ to be contiguous in the sequence length dimension. So we should either (1) transpose $V$ in GMEM as a pre-processing step, or (2) do an in-kernel transpose of tiles of $V$ after loading them into SMEM. We choose (2) here, the reason is in the paper.

There is another problem, the layout of FP8 and FP32 are very different, so we must transform the layout before loading the data into operand A register.

#### Accuracy

To reduce the numerical error of attention in FP8, we employ 2 techniques:

1. Block quantization
2. Incoherent processing
