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

The **limitations** are _no selective access, irreversible loss, output growth_. In order to solve such problems, we do propose new methods like gated residual, and advanced version such as mHC mentioned earlier.

Now lets dive into mHC. mHC is actually a improved version of HC (hyper connections), which applies more fine-grained control over residual connection by adding three matrix multiplication to residual, result of function and input. The formula becomes:

$$
h_{l}=\mathcal{H}_{l}^{\text{res}}h_{l-1}+\mathcal{H}_{l}^{\text{post}}f_{l-1}(\mathcal{H}_{l}^{\text{pre}}h_{l-1})
$$

What mHC improved is that they add more constraints to these matrices, so that has better mathematical theory support and achieved better score. So this method does not break the essence of traditional residual connections too.

Now let's talk about Attention Residuals, the formula becomes very different. For attention weights, we write them as $\alpha_{i\to l}=\phi(q_{l},k_{i})$ for a kernel function $\phi:\mathbb{R}^{d}\times \mathbb{R}^{d}\to \mathbb{R}_{\geq 0}$, where $q_{l}$ and $k_{i}$ are query and key vectors.

So the formula is:

$$
h_{l}=\alpha_{0\to l}\cdot h_{1}+\sum ^{l-1}_{i=1}\alpha_{i\to l}\cdot f_{i}(h_{i})
$$

The softmax attention is:

$$
\alpha_{i\to l}=\frac{\phi(q_{l},k_{i})}{\sum ^{l-1}_{j=0}\phi(q_{l},k_{j})}
$$

For each layer $l$, we define:

$$
q_{l}=w_{l},\quad k_{i}=v_{i}=
\begin{cases}
h_{1} &i=0 \\
f_{i}(h_{i}) & 1\leq i\leq l-1
\end{cases}
$$

where query $q_{l}=w_{l}$ is a layer-specific learnable parameter in $\mathbb{R}^{d}$. The input to layer $l$ is then:

$$
h_{l}=\sum ^{l-1}_{i=0}\alpha_{i\to l}\cdot v_{i}
$$

This is what we call **full attention residual**. The most important part of it, is that it has a multiplication between $f_{i}(h_{i})$ and itself, that's never occurred in traditional residual.

Traditional residual is just like RNN, and Attention Residual provides a new Transformer method across depth dimension.
