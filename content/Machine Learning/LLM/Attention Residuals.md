Paper link: http://arxiv.org/abs/2603.15031

In short, Attention Residual replaces the traditional identical residual mapping with a attention result of all previous layers. To reduce overhead, they group layers into blocks and apply block Attention Residuals, results in negligible overhead and better performance on same budget.

What makes me feel most interested is it's difference between [[Manifold-Constrained Hyper-Connections|mHC]], which is also a variant of residual.

Let's dive into the math essence of them.

**Notation.** Consider a batch of input sequence with shape $B\times T\times d$, where $B$ is the batch size, $T$ is the sequence length, and $d$ is the hidden dimension. For clarity, we write formulas for a single token: $h_{l}\in \mathbb{R}^{d}$ denotes the hidden state entering layer $l$, where $l\in\{1,\dots,L\}$ is the layer index and $L$ is the total number of layers. So the token embedding is $h_{1}$. The function $f_{l}$ represents the transformation applied by layer $l$. In Transformer models, we treat each self-attention and MLP as an individual _layer_.

For traditional **residual connection**, we have formula:

$$
h_{l}=h_{l-1}+f_{l-1}(h_{l-1})
$$

Expanding the recurrence, we have: $h_{l}=h_{1}+\sum ^{l-1}_{i=1}f_{i}(h_{i})$. The key insight behind residual connections is _identity mapping_: each layer preserves a direct path for both information (forward) and gradient (backward) to flow. During back propagation, we have:

$$
\frac{\partial \mathcal{L}}{\partial h_{l}}=\frac{\partial \mathcal{L}}{\partial h_{L}}\cdot \prod ^{L-1}_{j=l}\left( \mathbf{I}+ \frac{\partial \mathcal{f_{j}}}{\partial h_{j}}\right)
$$
