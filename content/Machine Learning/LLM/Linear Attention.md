Paper link: http://arxiv.org/abs/2006.16236
Traditional attention takes $O(N^{2})$ complexity, quadratic to sequence length, to compute, which is high time cost. So they proposed a method to change the attention from traditional _softmax_ attention to a feature map based dot product attention, which achieved linear complexity while has pretty good performance.

# Dive into Transformer

Let $x \in \mathbb{R}^{N\times F}$ denote a sequence of $N$ feature vectors of dimension $F$. A transformer is a function $T:\mathbb{R}^{N\times F}\to \mathbb{R}^{N\times F}$ defined by the composition of $L$ transformer layers like:

$$
T_{l}(x)=f_{l}(A_{l}(x)+x)
$$

$f(\cdot)$ represents feed forward network. $A(\cdot)$ represents self attention function, the $x$ stands for residual connection. And the self attention is the key part to change.
The self attention function $A(\cdot)$ computes, for every position, a weighted average of the feature representations of all the other positions with a weight proportional to a **similarity score** between the representations. In traditional attention, such score is computed with softmax, since we only need a similarity score, we can use some other ways to do so.

# Linearized Attention

They find that the only constraint they need to impose to $\text{sim}(\cdot)$ is to be non-negative. This includes all kernels $k(x, y):\mathbb{R}^{2\times F} \to \mathbb{R}^{+}$.
Given such a kernel with a feature representation $\phi(x)$ we can write the attention as:

$$
V'_{i}= \dfrac{\sum ^{N}_{j=1}\phi(Q_{i})\phi(K_{j})V_{j}}{\sum ^{N}_{j=1}\phi(Q_{i})\phi(K_{j})},
$$

and then further simplify it by making use of the associative property of matrix multiplication to:

$$
V'_{i}= \dfrac{\phi(Q_{i})\sum ^{N}_{j=1}\phi(K_{j})V_{j}}{\phi(Q_{i})\sum ^{N}_{j=1}\phi(K_{j})}.
$$

It is evident that the computational cost of original softmax attention scales with $O(N^{2})$, while their proposed _linear transformer_ has time and memory complexity $O(N)$ because they can compute the summation once and reuse them for every query. So it's like $O(N+N)$.

## More about complexity

For softmax attention, the total cost in terms of multiplications and additions scales as $O(N^{2}\max(D,M))$, where $D$ is the hidden dim of $Q,K$, $M$ is the hidden dim of $V$. On the contrary, for linear attention, we first compute the feature maps of dimensionality $C$. Subsequently, computing the new values requires $O(NCM)$ additions and multiplications.
You can think the dimension of feature maps like Taylor's Formula, so complete duplicate of exponential kernel would require infinite dimension. But for a linearized polynomial transformer of degree 2, the complexity is $O(ND^{2}M)$. This is usually good because $N$ nowadays are really big.
The authors use a degree 1 feature map of $\text{elu}(\cdot)$ in experiments with small context length.

## Causal Masking

This can be applied to linear attention easily too, by changing the summation of $N$ to $i$. This won't affect the linear complexity, because we can update such value with constant time in the process.

# Transformers are RNNs

With causal masking, any transformer layer can be written as a model that, given an input, modifies an internal state and then predicts an output, namely a Recurrent Neural Network.
The Transformer can be resulted into a RNN with 2 hidden states, namely the attention memory $s$ and the normalizer memory $z$.

$$
\begin{align*}
s_0 &= 0, \\
z_0 &= 0, \\
s_i &= s_{i-1} + \phi(x_i W_K)\,(x_i W_V)^T ,\\
z_i &= z_{i-1} + \phi(x_i W_K), \\
y_i &= f_l\!\left(
    \frac{ \phi(x_i W_Q)^T s_i }{ \phi(x_i W_Q)^T z_i } + x_i
\right).
\end{align*}
$$
