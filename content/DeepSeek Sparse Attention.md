Resource: [Paper](http://arxiv.org/abs/2512.02556)

**Prototype of DSA**. DSA has two components: a lightning indexer and a fine-grained token selection mechanism.

The **lightning indexer** computes the index score $I_{t,s}$ between the query token $\mathbf{h}_{t}\in \mathbb{R}^{d}$ and a preceding token $\mathbf{h}_{s}\in \mathbb{R}^{d}$, determining which tokens to be selected by the query token:

$$
I_{t,s}=\sum ^{H^{I}}_{j=1}w^{I}_{t,j}\cdot \text{ReLU}(\mathbf{q}^{I}_{t,j}\cdot \mathbf{k}^{I}_{s}),
$$

where $H^{I}$ denotes the number of indexer heads; $\mathbf{q}^{I}_{t,j}\in \mathbb{R}^{d^{I}}$ and $w^{I}_{t,j}\in \mathbb{R}$ are derived from the query token $\mathbf{h}_{t}$; and also does key (preceding) token. Use ReLU for throughput consideration, we still need to do attention.

Given the index scores $\{I_{t,s}\}$ for each query token $\mathbf{h}_{t}$, our **fine-grained token selection mechanism** retrieves only the key-value entries $\{\mathbf{c}_{s}\}$ corresponding to the top-k index scores. Then the attention output $\mathbf{u}_{t}$ is computed by applying the attention mechanism between the query token $\mathbf{h}_{t}$ and the sparsely selected key-value entries $\{\mathbf{c}_{s}\}$:

$$
\mathbf{u}_{t}=\text{Attn}(\mathbf{h}_{t}, \{\mathbf{c}_{s}\mid I_{t,s}\in \text{Top-k}(I_{t,:})\}).
$$
