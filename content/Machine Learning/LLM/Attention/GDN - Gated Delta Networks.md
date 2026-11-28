Paper link: [Gated Delta Networks: Improving Mamba2 with Delta Rule](http://arxiv.org/abs/2412.06464)

# Intro

In modern LLM, we use self-attention in Transformer architecture. The problem is self-attention scales quadratically with sequence length, leading to substantial computational demands.

To mitigate these issues, researchers explored alternatives like linear Transformer, which replace traditional softmax-based attention with kernel form dot-product-based [[Linear Attention]], substantially reducing memory requirements during inference, which also shows the essence of linear Transformer: a linear RNN with matrix-valued states.

Though simple naive linear attention performs worse than full attention, many methods and architectures are developed to improve it and achieves better and better performance. Such as incorporating data-dependent gating mechanisms akin to those in [[LSTM Explained|LSTM]], with models like GLA and [[Mamba]] as example.

In linear attention, the limitation is that the number of orthogonal key-value pairs they can store is _bounded by_ the model's dimension. When sequence length exceeds this dimension, "memory collisions" becomes inevitable, hindering exact retrieval.

Mamba2 address this limitation by introducing a simple _gated update rule_, $\mathbf{S}_{t}=\alpha_{t}\mathbf{S}_{t-1}+v_{t}k_{t}$, which uniformly decays all key-value associations at each step by a dynamic ratio, $\alpha_{t}\in(0,1)$. The problem of this method is coarse granularity. We have to update all key-value associations at the same time.

So the coming up solution is intuitive and straight forward, _gated delta rule_, it can update each key-value pair solely.

# Preliminary

Let's rewind the Mamba.

## Mamba2: Linear Attention with decay

**Algorithm.** It is known that the linear transformer can be formulated as linear recurrence when excluding normalization and query/key activations:

$$
\mathbf{S}_{t}=\mathbf{S}_{t-1}+v_{t}k_{t}^{\top}\in \mathbb{R}^{d_{v}\times d_{k}},\quad o_{t}=\mathbf{S}_{t}q_{t}\in \mathbb{R}^{d_{v}}
$$

Mamba made a improvement, which is a decay term:

$$
\mathbf{S}_{t}=\alpha_{t}\mathbf{S}_{t-1}+v_{t}k_{t}^{\top}\in \mathbb{R}^{d_{v}\times d_{k}},\quad o_{t}=\mathbf{S}_{t}q_{t}\in \mathbb{R}^{d_{v}}
$$

where $\alpha\in(0,1)$ is a data dependent scalar valued decay term that varies with $t$.

Above formula are single token prediction output, but in reality we need to batch all $L$ sequences together to increase throughput.

So we define a cumulative decay product $\gamma_{j}=\prod ^{j}_{i=1}\alpha_{i}$, and by expanding the recurrence, we can express it in a matrix parallel form:

$$
o_{t}=\sum ^{t}_{i=1}\left( \frac{\gamma_{t}}{\gamma_{i}}v_{i}k_{i}^{\top} \right)q_{t}=\sum ^{t}_{i=1}v_{i}\left( \frac{\gamma_{t}}{\gamma_{i}}k_{i}^{\top}q_{t} \right),\quad \mathbf{O}=(\left( \mathbf{Q}\mathbf{K}^{\top})\odot \Gamma \right)\mathbf{V}
$$

Here $\Gamma\in \mathbb{R}^{L\times L}$ is a decay aware causal mask where $\Gamma_{ij}=\frac{\gamma_{i}}{\gamma_{j}}$ if $i\geq j$ and $0$ otherwise.

**Chunkwise training.** Even though batched matrix parallel form training is available, still not enough, which motivates the use of chunkwise training. To summarize the chunkwise parallel form split inputs and outputs into several chunks of size $C$, and computes outputs for each chunk based on the final state of the previous chunk and query/key/value blocks of current chunk. Which is very similar to [[FlashAttention]] in my opinion, it is a hardware aware optimization but for linear attention.

The detailed formula can be found in the paper. Actually, it is much simpler than Flash Attention, since we only need deal with linear attention.

## Delta Network: Linear Attention with Delta Rule

The delta update rule _dynamically_ erases the value $v_{t}^{\text{old}}$ associated with the current input key $k_{t}$ and writes a new value $v_{t}^{\text{new}}$, which is performed with a linear combination of current input value and the old value based on the "writing strength" $\beta_{t}\in(0,1)$.

$$
\mathbf{S}_{t}=\mathbf{S}_{t-1}-(\mathbf{S}_{t-1}k_{t})k_{t}^{\top}+(\beta_{t}v_{t}+(1-\beta_{t})\mathbf{S}_{t-1}k_{t})k_{t}^{\top}=\mathbf{S}_{t-1}(\mathbf{I}-\beta_{t}k_{t}k_{t}^{\top})+\beta_{t}v_{t}k_{t}^{\top}
$$

The theoretical explanation and mathematical proof is not listed in this paper, might be checked in the original paper.

In the past this method is not welcomed though it has better performance due to slow train speed, but now we have chunkwise train method to alleviate this problem, the chunkwise form can be checked in the paper.

# Gated Delta Networks

Now lets dive into the key topic, GDN.

## Formulation: Gated Delta Rule

The proposed gated delta rule is simple yet effective:

$$
\mathbf{S}_{t}=\mathbf{S}_{t-1}(\alpha_{t}(\mathbf{I}-\beta_{t}k_{t}k_{t}^{\top}))+\beta_{t}v_{t}k_{t}^{\top}
$$

where data dependent gating term $\alpha_{t}\in(0,1)$ controls state decay. It combines the benefit of both methods: the gating term enables adaptive memory management, while the delta update structure facilitates effective key-value association learning.

The learning objective of GDN is:

$$
\|\mathbf{S}_{t}-\alpha_{t}\mathbf{S}_{t-1}\|^{2}_{F}-2\langle\mathbf{S}_{t}k_{t},\beta_{t}(v_{t}-\alpha_{t}\mathbf{S}_{t-1}k_{t})\rangle
$$

The first term represents how well does the new state remembers the decayed old state, which is called _memory retention_. The seconded term tells how well current state learned about the new value. Why it has such meaning? First the inner product tells similarity, and $\mathbf{S}_{t}k_{t}$ tells us when we query current state with this key, what value we get, and $\beta_{t}(v_{t}-\alpha_{t}\mathbf{S}_{t-1}k_{t})$ gives us the new knowledge we write into current state, which is the difference between new value and decayed also remembered old value. Think about it.

## Case Study

They test DeltaNet, Mamba2 and GDN with S-NIAH, a memory retention benchmark. A key-value pair acts like a needle in the long context (haystack), and the model must recall the value when given the key.

Conclusions are:

- Decay hurts memory retention
- Gating facilitates filtering
- Delta rule helps memorization

## Algorithm: Chunkwise training

They also developed chunkwise version of GDN, it is too long to be written here.

# Experiments

They proposed two kinds of architectures, first one is combining linear recurrent layers with sliding window attention (SWA), second one is stacking Mamba2, GDN and SWA.

The detail architecture of GDN block is the following:

![[Pasted image 20260402160911.png]]

# Conclusion

In this work, they introduced Gated DeltaNet, which enables better key-value association learning compared to Mamba2 and more adaptive memory clearance than DeltaNet, leading to consistently better empirical results across various tasks. They extended the parallel algorithm to enable hardware-efficient training of Gated DeltaNet. The hybrid Gated DeltaNet model achieves even higher training throughput and overall performance, making it well-suited for practical deployment.

GDN is also used in modern Qwen3.5 series.
