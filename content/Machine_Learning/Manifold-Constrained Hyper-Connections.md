Paper link: http://arxiv.org/abs/2512.24880

# Introduction

**Hyper-Connections** was introduced to replace **residual connection** in order to apply more control over the information passed down so that can improve model performance. From the result, we can see that it did it, but sacrificed training stability. This is because hyper-connections has no limit over the control. Let's examine the detail.

# Related Work

## Instability

Hyper connections just replace residual connections, which originally looks like this:

$$
\mathbf{x}_{l+1}=\mathbf{x}_{l}+\mathcal{F}(\mathbf{x}_{l},\mathcal{W}_{l})
$$

The HC version is defined as:

$$
\mathbf{x}_{l+1}=\mathcal{H}^{\text{res}}_{l}\mathbf{x}_{l}+{\mathcal{H}^{\text{post}}_{l}}^{\top}\mathcal{F}(\mathcal{H}^{\text{pre}}_{l}\mathbf{x}_{l},\mathcal{W}_{l})
$$

Formally, HC computes the coefficients as follows:

$$
\begin{cases}
\tilde{\mathbf{x}}_{l}=\text{RMSNorm}(\mathbf{x}_{l}) \\
\mathcal{H}^{\text{pre}}_{l}=\alpha^{\text{pre}}_{l}\cdot \text{tanh}(\theta^{\text{pre}}_{l}\tilde{\mathbf{x}}^{\top}_{l})+\mathbf{b}^{\text{pre}}_{l} \\
\mathcal{H}^{\text{post}}_{l}=\alpha^{\text{post}}_{l}\cdot \text{tanh}(\theta^{\text{post}}_{l}\tilde{\mathbf{x}}^{\top}_{l})+\mathbf{b}^{\text{post}}_{l} \\
\mathcal{H}^{\text{res}}_{l}=\alpha^{\text{res}}_{l}\cdot \text{tanh}(\theta^{\text{res}}_{l}\tilde{\mathbf{x}}^{\top}_{l})+\mathbf{b}^{\text{res}}_{l},
\end{cases}
$$

So we can conclude that HC has no constraints on the transforms with input signal. It may lead to unbounded signal amplification or attenuation, result in instability during large-scale training.

## System Overhead

HC also adds a lot of memory access costs, which constitutes one of the primary bottlenecks in modern model architectures. We need to calculate additional parameters and increases memory cost by a factor approximately proportional to $n$. Further more, HC requires $n$-fold more communication cost in pipeline parallelism, leading to larger bubbles and decreasing the training throughput.

# Method

## Manifold-Constrained Hyper-Connections

Drawing inspiration from the identity mapping principle of residual connection, the core premise of $m$HC is to constrain the residual mapping $\mathcal{H}^{\text{res}}$ onto a specific manifold. The original identity mapping ensures stability by enforcing $\mathcal{H}^{\text{res}}_{l}=\mathbf{I}$, it precludes information exchange with residual stream, which is critical for maximizing the potential of multi-stream architectures. Therefore, we can maintain signal propagation stability and also preserve model's expressiveness. The solution is to restrict $\mathcal{H}^{\text{res}}_{l}$ to be a **_doubly stochastic matrix_**, which has _non-negative entries where both the rows and columns sum to 1_. Formally, let $\mathcal{M}^{\text{res}}$ denote the manifold of doubly stochastic matrices (also known as the Birkhoff polytope). We constrain $\mathcal{H}^{\text{res}}_{l}$ to $\mathcal{P}_{\mathcal{M}^{\text{res}}}(\mathcal{H}^{\text{res}}_{l})$, defined as:

$$
\mathcal{P}_{\mathcal{M}^{\text{res}}}(\mathcal{H}^{\text{res}}_{l}) \coloneqq \{\mathcal{H}^{\text{res}}_{l}\in \mathbb{R}^{n\times n}\mid \mathcal{H}^{\text{res}}_{l}\mathbf{1}_{n}=\mathbf{1}_{n},\mathbf{1}_{n}^{\top}\mathcal{H}^{\text{res}}_{l}=\mathbf{1}_{n}^{\top},\mathcal{H}^{\text{res}}_{l}\geq 0\},
$$

when $n=1$, the doubly stochastic condition degenerates to the scalar 1, thereby recovering the original identity mapping. The choice of double stochasticity confers to several rigorous theoretical properties beneficial for large-scale model training:

1. **Norm Preservation**: The spectral norm of a doubly stochastic matrix is bounded by 1. This implies that the learnable mapping is non-expansive.
2. **Compositional Closure**: The set of doubly stochastic matrix is closed under matrix multiplication, thereby preserving stability throughout the entire depth of the model.
3. **Geometric Interpretation via the Brikhoff Polytope**: The set $\mathcal{M}^{\text{res}}$ forms the Birkhoff polytope, which is the convex hull of the set of permutation matrices. This provides an interpretation that the residual mapping acts as a convex combination of permutations. Mathematically such repetition tends to increase the mixing of information across streams monotonically.

The non-negativity also prevents signal cancellation between positive and negative coefficients.

## Parameterization and Manifold Projection

$m$HC did some modification to the calculation process. Given the input hidden matrix $\mathbf{x}_{l}\in \mathbb{R}^{n\times C}$ at the $l$-th layer, we first flatten it into a vector $\vec{\mathbf{x}}_{l}=\text{vec}(\mathbf{x}_{l})\in \mathbb{R}^{1\times nC}$ to preserve full context information. Then do the following mapping:

$$
\begin{cases}
\vec{\mathbf{x}}_{l}'=\text{RMSNorm}(\vec{\mathbf{x}}_{l}) \\
\tilde{\mathcal{H}}_{l}^{\text{pre}}=\alpha_{l}^{\text{pre}}\cdot(\vec{x}_{l}'\varphi_{l}^{\text{pre}})+\mathbf{b}_{l}^{\text{pre}} \\
\tilde{\mathcal{H}}_{l}^{\text{post}}=\alpha_{l}^{\text{post}}\cdot(\vec{x}_{l}'\varphi_{l}^{\text{post}})+\mathbf{b}_{l}^{\text{post}} \\
\tilde{\mathcal{H}}_{l}^{\text{res}}=\alpha_{l}^{\text{res}}\cdot \text{mat}(\vec{x}_{l}'\varphi_{l}^{\text{res}})+\mathbf{b}_{l}^{\text{res}}, \\
\end{cases}
$$

where $\varphi$ are linear projections for dynamic mapping and $\text{mat}(\cdot)$ is reshape function from $\mathbb{R}^{1\times n^{2}}$ to $\mathbb{R}^{n\times n}$.
Then the final constrained mapping are obtained via:

$$
\begin{cases}
\mathcal{H}^{\text{pre}}=\sigma(\tilde{\mathcal{H}}_{l}^{\text{pre}}) \\
\mathcal{H}^{\text{post}}=2\sigma(\tilde{\mathcal{H}}_{l}^{\text{post}}) \\
\mathcal{H}^{\text{res}}_{l}=\text{Sinkhorn-Knopp}(\tilde{\mathcal{H}}_{l}^{\text{res}}),
\end{cases}
$$

where $\sigma(\cdot)$ denotes the Sigmoid function. The $\text{Sinkhorn-Knopp}(\cdot)$ operator firstly makes all elements to be positive via an exponent operator and then conducts **iterative** normalization process that alternatively rescales rows and columns to sum to 1. They choose 20 iterations as a practical value in experiments.
