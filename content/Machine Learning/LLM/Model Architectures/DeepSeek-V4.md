# 1. Abstract

DeepSeek-V4 released two new MoE models, both supporting a context length of one million tokens. The key upgrades and optimizations are:

- A hybrid attention architecture that combines Compressed Sparse Attention (CSA) and Heavily Compressed Attention (HCA) to improve long-context efficiency
- [[Manifold-Constrained Hyper-Connections]] to enhance conventional residual connections
- Muon optimizer for faster convergence and greater training stability

# 2. Architecture

V4 retains the Transformer architecture and Multi-Token Prediction (MTP) modules, while introducing several key upgrades over V3 just like mentioned in abstract.

## 2.1. Designs Inherited from DeepSeek-V3

**Mixture-of-Experts.** V4 still adopt the [[DeepSeekMoE]] for feed forward network, but change the activation function that computes the affinity scores from $\text{Sigmoid}(\cdot)$ into $\text{Sqrt}(\text{Softplus}(\cdot))$. Also employ the [[DeepSeek-V3|auxiliary-loss-free]] strategy, augmented by a slight sequence-wise balance loss.

For V4, the constraint on the number of routing target nodes is removed, also the parallelism strategy is redesigned. Furthermore, compared with V3, they replace the dense FFN layers in the initial several Transformer blocks with MoE layers that employ Hash routing.

**Multi-Token Prediction.** Same strategy as V3.

## 2.2. Manifold Constrained Hyper-Connections

Just check my previous blog [[Manifold-Constrained Hyper-Connections]].

## 2.3. Hybrid Attention

Current trend on LLM is agent, which means we need longer and longer context length to deal with larger and more complex task. So 1 million context length is kind of a common suit now, and applying full attention even with super optimized kernel won't be realistic, so we need to have new attention design in order to mitigate the computational bottleneck.

They designed two efficient attention architectures Compressed Sparse Attention (CSA) and Heavily Compressed Attention (HCA).

### 2.3.1. Compressed Sparse Attention

The core structure was shown in the following picture:

![[Pasted image 20260425172842.png]]

**Compressed Key-Value Entries.** Let $H\in \mathbb{R}^{n\times d}$ be a sequence of input hidden states, where $n$ is the sequence length and $d$ is the hidden size. CSA first compute two series of KV entries $C^{a},C^{b}\in \mathbb{R}^{n\times c}$ and their corresponding compression weights $Z^{a},Z^{b}\in \mathbb{R}^{n\times c}$, where $c$ is the head dimension. Next, each $m$ KV entries in $C^{a}$ and $C^{b}$ will be compressed into one entry according to their compression weights and learnable positional biases $B^{a},B^{b}\in \mathbb{R}^{m\times c}$, producing $C^{\text{Comp}}\in \mathbb{R}^{\frac{n}{m}\times c}$.

Each compressed entry $C_{i}^{\text{Comp}}\in \mathbb{R}^{c}$ is computed by

$$
\begin{align}
	[S^{a}_{mi:m(i+1)-1};S^{b}_{m(i-1):mi-1}]&=\text{Softmax}_{\text{row}}([Z^{a}_{mi:m(i+1)-1}+B^{a};Z^{b}_{m(i-1):mi-1}+B^{b}]), \\
	C_{i}^{\text{Comp}} &=\sum_{j=mi}^{m(i+1)-1}S_{j}^{a}\odot C_{j}^{a}+ \sum ^{mi-1}_{j=m(i-1)} S_{j}^{b}\odot C_{j}^{b}
\end{align}
$$

where $\odot$ is element wise matrix multiplication. We can notice that the indices for $a$ and $b$ is different, $a$ is one step forward than $b$, and $C^{\text{Comp}}_{i}$ is made up of both of them which makes the compressed result can perceive both information from current state and previous state, that is the benefit. The reason why we need to have two series of KV entries is this.

Currently, we have compressed the sequence length to $\frac{1}{m}$ times.

**Lightning Indexer for Sparse Selection.** After obtaining the compressed KV entries $C^{\text{Comp}}$, CSA applies [[DeepSeek Sparse Attention|DSA]] strategy to select top-k compressed KV entries for core attention.

**Shared Key-Value MQA.** After selecting the sparse KV entries, CSA performs core attention in a Multi-Query Attention manner, where each KV entry in $C^{\text{SprsComp}}_{t}$ serves as both attention key and value. It is a MQA here because we can see that our compressed KV has only 1 head each from previous content.

**Grouped Output Projection.** The output of core attention will have size $\mathbf{o}_{t}\in \mathbb{R}^{cn_{h}}$ where $c$ is head dimension and $n_{h}$ is head number. And we will need to project it to $d$ dimension hidden state, if we do it naively, it will impose a substantial computational burden. To mitigate this cost, they designed a grouped output projection strategy. First split $n_{h}$ outputs into $g$ groups and for each group project them to $d_{g}$ and finally concatenate them back to $d$.

### 2.3.2. Heavily Compressed Attention
