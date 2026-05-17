In modern parallel training methods, there are three methods that partitions tensors: Tensor Parallelism (TP), Sequence Parallelism (SP) and Context Parallelism (CP). We will introduce these parallelism in this article.

A common transformer activation tensor often looks like this:

$$
	X\in \mathbb{R}^{B\times S\times H}
$$

where $B$ is batch size, $S$ is sequence length and $H$ is hidden dimension. Different parallelism splits different dimensions. Like classic data parallelism splits batch dimension so that we can train more data simultaneously.

# Tensor Parallelism

TP splits **_model parameters or you can say hidden dimension_** across machines.

For calculations like $Y=XW$ where $W\in \mathbb{R}^{H\times 4H}$ (linear layer has 4 times hidden dimension is kind of a tradition?), if we have 4 machines in use, each machine stores only part of $W$ like $W=[W_{1},W_{2},W_{3},W_{4}]$ and each machine computes $Y_{i}=XW_{i}$ where $W_{i}\in \mathbb{R}^{H\times H}$. In the end, all results are gathered.

So TP can reduce the parameter size required on each machine but add more communication overhead, you an imagine that after each MLP or projection we need to do such gather.

# Sequence Parallelism

It splits sequence length dimension just like its name.

So instead of $X\in\mathbb{R}^{B\times S\times H}$ we have $X_{i}\in \mathbb{R}^{B\times S/P\times H}$ now, where $P$ is the number of machines. Each machine is only responsible for part of the tokens now.

Why do we have SP? This is because there are some operations that operates over sequence dimension and will store duplicate activations across all of the machines (like LayerNorm, Residual). So sequence parallelism is actually splitting activations to reduce space and compute cost (only needs to compute part of the activation too).

It splits on different axis with TP, so they can be used together.

SP also requires synchronization. But less that TP.

# Context Parallelism

CP also splits the sequence dimension, but the purpose is different. It is designed to solve the problem with long context attention quadratic complexity.

It parallels **attention context**, but attention computation needs access to all other tokens, so machines must exchange KV caches and tokens. The communication usually happens with ring communication, all-gather or P2P exchange.

Since it is designed for attention computation, it is also independent from SP, and we can apply DP + PP + SP + TP + CP in distributed modern training.
