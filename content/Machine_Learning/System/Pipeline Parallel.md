Naive pipeline parallel (model parallel) has many disadvantages:

1. Low GPU utilization: at any point of time, only one device is working, others are idle.
2. No interleaving of computation and communication: while sending intermediate results to next device, GPUs are idle.
3. High memory demand: first GPU needs to store all activations until the whole batch completes.

To solve such kind of problems, we introduce various methods.

# Bubbles

We divide input data into smaller micro-batches, so that we can overlap forward pass of each worker, because they can finish 1 micro batch and send it then keep on working, this also works for backward pass. Such technique reduces the size of idle time, which is usually called bubble.

# Memory Shortage

## Gradient Checkpoint

The storage of activations for backward demands huge memory resource, for _vanilla_ back propagation we store activations in each node, the memory cost is $O(N)$ and compute cost is $O(N)$. If we do _recompute_ for each node, then the memory cost is $O(1)$ and compute cost is $O(N^{2})$.

So here comes the classic scenario of computer science, we have two extreme, one in time and one in space, then there will appear a fusion of both. Which is the _gradient checkpoint_ we are going to introduce.

We cache the activations of every $\sqrt{ N }$ layers, so the memory cost becomes $O(\sqrt{ N })$ and compute cost is $O(\sqrt{ N }^{2})$ which is just $O(N)$.

## Immediate Backward

This is also a method that saves activation memory, it is called 1F1B flush, which means we start backward as soon as possible. For the device that _handles last stage_, when it finished forward, it will do backward immediately, so that earlier stages can do backward sooner and free the memory.

Its benefit is that it reduces memory cost for storing the activations, since the number of micro-batches in flight is smaller.

## Further Improvement

Currently we are only dividing the model according to the number of devices, but actually we can divide the model even smaller, so that the devices forms a kind of ring. For example, we have 4 devices and chunk the model into 8 parts, then device 1 is responsible for layer 1 and 5, etc. In this case, each batch will run through device 1 to 4 and then send back to device 1 to keep on doing the forward, which enables more overlapping so that the bubble can become smaller.
