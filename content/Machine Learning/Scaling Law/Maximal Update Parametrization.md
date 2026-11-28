# What?

Paper link: http://arxiv.org/abs/2203.03466

Maximal Update Parametrization ($\mu$P), a method that makes many optimal Hyper Parameters (HP) remain stable even as model size changes. Which leads to a new HP tuning paradigm we call $\mu$_Transfer_: Parameterize the target model in $\mu$P, tune the HP indirectly on a smaller model, and _zero-shot transfer_ them to the full-sized model.

![[muP lr curve.png]]

# Why?

HP tuning is critical to deep learning. But many published baselines are hard to compare to one another due to varying degrees of HP tuning. The HP tuning in deep learning is also an expensive process, especially for current large language models. So we want to apply the idea of scaling law that is transferring the HP of small models to bigger ones.

# How?

## Parametrization Matters: A Primer

In this section, we give a basic primer on why the correct parametrization can allow HP transfer across width.

The Central Limit Theorem (CLT) says that, if $x_{1},\dots,x_{n}$ are iid (Independent and Identically Distributed) samples from a zero-mean, unit-variance distribution, then $\frac{1}{\sqrt{ n }}(x_{1}+\dots +x_{n})$ converges to a standard Gaussian $\mathcal{N}(0,1)$ as $n\to \infty$. Therefore we can say that $\frac{1}{\sqrt{ n }}$ is the right order of _scaling factor_ $c_{n}$ such that $c_{n}(x_{1}+\dots+x_{n})$ converges to something nontrivial.

Why this means we can transfer from smaller models to bigger ones?

Suppose we are going to minimize the function:

$$
F_{n}(c)=\mathbb{E}_{x_{1},\dots ,x_{n}}f(c(x_{1}+\dots+x_{n}))
$$

over $c\in \mathbb{R}$, for some bounded continuous function $f:\mathbb{R}\to \mathbb{R}$. You can consider $x_{i}$ as randomly initialized parameters of a width-$n$ neural network and $c$ as a HP such as learning rate, and $f$ is the model performance since we are discussing the relationship between HP and trained model performance. If we reparametrize $c=\frac{\alpha}{\sqrt{ n }}$ for $\alpha\in \mathbb{R}$ Then by CLT, $G_{n}(\alpha)=F_{n}(c)=\mathbb{E}f(\mathcal{N}(0,\alpha^{2}))$ stabilizes into a function of $\alpha$ as $n\to\infty$. Then for sufficiently large $n$, the optimal $\alpha^{*}_{n}=\arg\min_{\alpha}G_{n}(\alpha)$ should be close to $\alpha^{*}_{N}$ for any $N>n$, and indeed, for $N=\infty$. This precisely means we can _transfer_ the optimal $\alpha_{n}^{*}$ for a smaller problem to a larger problem: $G_{N}$ is approximately minimized by $\alpha^{*}_{n}$ and $F_{N}$ is approximately minimized by $c^{*}_{n}\sqrt{ \frac{n}{N} }$. We say the parametrization $c=\frac{\alpha}{\sqrt{ n }}$ is the _correct parametrization_ for this problem. If we parametrize the learning rate and other HPs correctly, then we can directly copy the optimal HPs for a narrower network into a wide network and expect approximately optimal performance, this is what _zero-shot transfer_ mean.

We emphasize that, to ensure transferability of any HP, it's not sufficient to reparametrize _only_ that HP, but rather we need to identify and correctly reparametrize _all_ HP in the table below:

| Optimizer Related                                         | Initialization           | Parameter Multipliers                              |
| --------------------------------------------------------- | ------------------------ | -------------------------------------------------- |
| learning rate (LR), momentum, Adam beta, LR schedule, etc | per-layer init. variance | multiplicative constants after weights/biases, etc |

The $\mu$P for General Neural Networks looks like this:

|            | Input weights & all biases | Output weights           | Hidden weights       |
| ---------- | -------------------------- | ------------------------ | -------------------- |
| Init. Var. | $1 / \text{fan\_in}$       | $1 / \text{fan\_in}^{2}$ | $1 / \text{fan\_in}$ |
| SGD LR     | fan_out                    | $1 / \text{fan\_in}$     | 1                    |
| Adam LR    | 1                          | $1 / \text{fan\_in}$     | $1 / \text{fan\_in}$ |

fan_in and fan_out just means input dimension and output dimension. In general, the three columns here can be interpreted as linear layers that have {finite, infinite, infinite} input dimension and {infinite, finite, infinite} output dimension in an infinite-width neural network.

## Further explanation of the $\mu$P tables

We can classify any dimension in a neural network as "infinite" if it scales with width, or "finite" otherwise. For example in Transformer, $d_{\text{model}},d_{\text{ffn}},d_{\text{head}},n_{\text{head}}$ are all infinite, but vocab size and context length are finite. Then we can categorize parameter tensors by how many infinite dimensions they have. If there are two such dimensions, then we say the parameter is _matrix-like_; if there is only one, we say it is _vector-like_; if there is none, we say it is _scalar-like_.

We have an Alternative (Equivalent) $\mu$P Formulation for Easier Implementation, its advantage is that it gives a uniform scaling rule of initialization and learning rate for all vector-like parameters (input weights & biases and output weights).

|            | Input weights & all biases | Output weights       | Hidden weights       |
| ---------- | -------------------------- | -------------------- | -------------------- |
| Init. Var. | $1 / \text{fan\_in}$       | 1                    | $1 / \text{fan\_in}$ |
| Multiplier | 1                          | $1 / \text{fan\_in}$ | 1                    |
| SGD LR     | fan_out                    | fan_in               | 1                    |
| Adam LR    | 1                          | 1                    | $1 / \text{fan\_in}$ |

You may wonder, hey! The init. var. of input weights & biases is different from output weights why you say they are uniform? Because we say they are uniform by they are both **independent of the width** now. In the previous table, the fan\*in of output weights is width of the model.

But how can we do this? It is because we introduce the _Multiplier_, it transforms our computation from $y=Wx$ into $y=\alpha Wx$, the $\alpha$ here is the multiplier.

To explain the details clearer, let's go through an example. Let $f*{t}(\xi)$ denote the neural network function after $t$ steps of training and evaluated on input $\xi$. Consider a parameter $W$ with learning rate $C$, initialized as $W\sim \mathcal{N}(0, B^{2})$, and with a multiplier $A$. Then for arbitrary $\theta>0$, $f_{t}(\xi)$ stays the same for all $t$ and $\xi$ if we set

$$
A\leftarrow A\theta,B\leftarrow \frac{B}{\theta},C\leftarrow \frac{C}{\theta}
$$

when the optimizer is Adam. (SGD case is similar)

Such re-scaling derives from pure math. We have $y=AWx$, after re-scaling, we initialize $\tilde{W}=\frac{W}{\theta}$, so we multiply $A$ by $\theta$, then the variance and result stays the same, the **forward pass is unchanged**.

For backward pass, let

$$g= \frac{\partial L}{\partial W}$$

by chain rule

$$g=A \frac{\partial L}{\partial y}x^{\top}$$

after re-scaling

$$\tilde{g}= \frac{\partial L}{\partial \tilde{W}}=g\cdot\theta$$

In Adam update:

$$
\Delta W=C \frac{m}{\sqrt{ v }+\varepsilon}
$$

where $m\sim g,v\sim g^{2}$, under $\tilde{g}=\theta g$, $\frac{m}{\sqrt{ v }}$ stays the same.

But we have $\tilde{W}= \frac{W}{\theta}$, so $\Delta\tilde{W}= \frac{\Delta W}{\theta}$, so we set $C=\frac{C}{\theta}$ for Adam.

In the previous table, multiplier is 1, when we plug in $\theta= \frac{1}{\text{fan\_in}}$ we will get the data in second table.
