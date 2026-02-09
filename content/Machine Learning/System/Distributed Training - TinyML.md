Models are too big, so we need multiple devices to train them.

# Strategies for Scalable Training

## Data Parallelism

Same model on $N$ GPU, split the Training Dataset across them.
Two different _roles_ in framework:

- **Parameter Server**: receive gradients from workers and send back the aggregated results
- **Workers**: compute gradients using split dataset and send to parameter server
  Compare with single node training, DP has two more synchronization steps:

```
For iteration i in T:
	1. Replicate / Pull gradients form parameter server
	2. Sample data from datasets x(i, j), y(i, j)
	3. Compute gradients w(i, j)
	4. Push gradients to parameter server and sum
	5. Perform gradient step: w(i, j) = w(i, j) - lr * avg(w(i))
```

### Communication Primitives

- **Send**: send from $n_{0}$ to $n_{1}$
- **Receive**: receive from $n_{1}$ to $n_{0}$
- **Scatter**: split a tensor to all other workers, e.g. $[1,2,3,4]$ to $[1],[2],[3],[4]$
- **Gather**: the reverse of scatter
- **Reduce**: a bit different from gather, e.g $[1],[2],[3],[4]$ to $[1+2+3+4]=[10]$
- **Broadcast**: send identical copies to all other workers
- **All-Reduce**: perform reduce on all workers
- **All-Gather**: perform gather on all workers

### Use the primitives

So how can we describe the communication schemes used in DP? Broadcast then Reduce! But if we check the bandwidth requirement we will find out that each worker demands $O(1)$ while Parameter Server requires $O(N)$, this can be a **bottleneck**.

Can we perform the aggregation without a central server? **ALL-REDUCE!** However naive All-Reduce will require $O(N)$ time and bandwidth on all workers, but we have better implementations like **Ring** and **Parallel Reduce**. Ring has $O(N)$ time and $O(1)$ bandwidth, Parallel Reduce has $O(1)$ time and $O(N^2)$ bandwidth. There is also a method called **Recursive Halving** which achieves $O(\log N)$ time.

#### Ring All Reduce

In **ring** all reduce, for example, if we want to aggregate the gradients, for $N$ workers we can separate all gradients into $N$ packs, and each worker passes their corresponding part to the next worker, after that, each worker passes the aggregated gradients and keeps doing this, which forms a ring-like trajectory.

In the end, we do a **all gather** to make every worker has all of the gradients. In this case, each worker's load is only $B$ for gradient size $B$, rather than $NB$ for naive all reduce.

### DeepSpeed - Reducing Memory

Even the best GPU cannot fit the model weights of current large language models, so we have DeepSpeed ZeRO-1/2/3 which proposes to partition **optimizer states, gradients and weights**, so that each GPU only needs to store part of the data. In PyTorch, ZeRO-3 is implemented via `FullyShardedDataParallel`, known as FSDP.

## Pipeline Parallelism

Single copy of data, but split the model across the GPU.

We can imagine the naive implementation be like, we have 4 GPU, and they run forward 1 to 4 sequentially, then backward 4 to 1 sequentially, this works but will have huge amount of idle time, the utilization will be very low.

How to improve? We can separate the batch into micro batches: $[16,10,512]\to 4*[4,10,512]$ this improves more working overlap, because when the worker finished a micro batch and was sending it, it can keep processing the next micro batch, which elevates utilization from $25\%$ to around $57\%$.

## Tensor Parallelism

Single copy of data, split the activations then synchronize.
Even with optimization, PP still has a lot of idle time, we can keep improving this by making the model partition **more fine-grained**. Which leads to Tensor Parallelism, split a weight tensor into $N$ chunks and paralleled.

## Sequence Parallelism

Suppose we have long text for training and it looks like: $[\text{Batch Size, Tokens, Hidden Dimension}]$.

Data parallelism splits batch size, while sequence parallelism splits tokens.

Sequence Parallelism in fully connected layers is just like Data Parallelism, but in attention layer it becomes different, because the computation is quadratic about sequence length. We need all to all communication to solve this.

### Solution 1: Head partition

We will have head $1,\dots ,d$ of $Q,K,V$ about the partitioned token in each GPU. So we can only compute one meaningful result for each head, in total we can get $N$ attention. But we need to recompose them, and that cost is huge. We propose all to all communication to get all $Q,K,V$ about one head in one GPU, so in the GPU itself it can compute all attention of this head, and greatly reduced communication cost. Think it like arranging $Nd$ compare to arranging $N^{2}$ objects.

### Solution 2: Ring Attention

After each computation, we switch $K,Q$ with each other, so after a round, we can compute all the attention, and this also only needs to arrange $Nd$ objects.

## Hybrid Parallelism

We can combine the methods introduced above to make better performance and be capable to train even larger models.

### Auto-Parallel

How to choose the best parallel strategy given so many methods on such big models?

Alpa is a unified compiler for distributed training which can surpass human best manual systems. It will create a Alpa Hierarchical Space based on computational graph's whole search space, and search for the best strategy.

# Bottlenecks of Distributed Training

The latency mainly comes from three aspects: memory bandwidth, computation and network bandwidth. The network bandwidth or latency is usually the biggest bottleneck in distributed training. Then the second is usually memory bandwidth.

## Reduce Transfer Data Size

### Gradient Compression

#### Deep Gradient Compression

Our main purpose now is to reduce memory load, we can **reduce transfer data size** by **pruning** and **quantization**, pruning is kind of **sparse communication**, one of the limitation is **performance degradation**. This happens because when we are using sparse communication, the pruned gradients are accumulated in local buffers so that we can perform somewhat batch update. If we use the accumulated gradient directly it won't match up with the optimizer's momentum, so what we should really accumulate is the velocity.

Another technique is called warm up training. In the **early stages** of training, the network is **changing rapidly**, so local gradient accumulation will **aggravate** the problem. We can improve this by warming up learning rate and sparsity, which means start with a low value and increase them in each epoch.

#### PowerSGD: Low-Rank Gradient Compression

Instead of using fine-grained pruning, adapt low-rank factorization instead.

### Gradient Quantization

#### 1-bit SGD

We save a 32-bit quantization error **locally**, and only pass a 1-bit quantized gradient, this reduces memory bandwidth a lot and only increases a little bit of memory usage.

#### Threshold Quantization

Choose a hyper parameter $\tau$ stands for the threshold, and we quantize the gradient to $\tau,-\tau,0$ for $\geq \tau, \leq -\tau$ and in between. We also have a error matrix.

#### TernGrad - Ternary

Quantize $q_{i}=\text{sign}(g_{i})\times\text{Bernoulli}\left(  \frac{|g_{i}|}{\max(g)}  \right)$, so we have a matrix of $1,0,-1$ now, we scalar it with $\max(g)$ to makes the expectation of gradient stay the same. No error is needed in this case.

## Delayed Gradient Update: overcome the Latency Bottleneck

When the latency is high, we need to wait for a long time or even longer than computation time to get all the updated parameters. We can solve this by delaying the update, with delay the workers can keep performing local updates while the parameters are in transmission.

Now we update the gradients like:

$$
w_{(i,j)}=w_{(i,j)}-\eta(\nabla w_{(i,j)}-\nabla w_{(i-D,j)}+\overline{\nabla w_{(i-D)}})
$$

This form deals with the staleness of delayed parameters.
