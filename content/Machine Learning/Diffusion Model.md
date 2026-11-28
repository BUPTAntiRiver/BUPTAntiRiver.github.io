Resource: [lec5_diffusion.pdf](https://mit-6s978.github.io/assets/pdfs/lec5_diffusion.pdf)

We are going to talk about four parts about diffusion model in this little article:

- Forward process: add noise to data
- Reverse process: learn to denoise
- Training objective: from Hierarchical VAE to L2 loss
- Noise Conditional Network: represent distributions

# What is Noise?

Adding Gaussian noise to a data point equals to sampling $x\sim\mathcal{N}(x\mid x_{0},\sigma)$, for data distribution, we are doing a convolution (of PDF) between $p_{\text{data}}(x)$ and normal distribution.

The process of adding noise is like doing convolution repeatedly until it becomes a noise distribution.

# Forward Process

From initial distribution $x_{0}$ to final noisy distribution $x_{T}$, the distribution passed a lot of steps. In each step we have:

$$
x_{t} = \sqrt{ 1-\beta_{t} } x_{t-1}+\sqrt{ \beta_{t} } \epsilon,\quad \epsilon\sim\mathcal{N}(0,\mathbf{I})
$$

The coefficients are designed to preserve variance. $t$ i called "schedule", key to Diffusion Model's success. Former part is mean of $x_{t}$, later part is std of $x_{t}$. So we can rewrite the equation as a conditional distribution:

$$
q(x_{t}\mid x_{t-1})=\mathcal{N}(x_{t}\mid \sqrt{ 1-\beta_{t} }x_{t-1},\beta_{t}\mathbf{I})
$$

If we concatenate transitions from $x_{0}$ to $x_{t}$:

$$
\begin{align*}
q(x_{t}\mid x_{0})&=\mathcal{N}(x_{t}\mid \sqrt{ \bar{\alpha}_{t} }x_{0}, (1-\bar{\alpha}_{t})\mathbf{I}) \\
\text{where } \alpha_{t} &\coloneqq 1-\beta_{t}\\
\bar{\alpha}_{t}&\coloneqq \prod ^{t}_{s=1}\alpha_{s}
\end{align*}
$$

$\beta_{t}$ is some function about $t$, it might be linear or cosine, etc.

**In short**, forward process has predefined conditional distributions (schedule $\beta$), Gaussian with controllable mean and std, _divide_ and conquer strategy (time step)

# Reverse Process

## Theorem

We use a parameterized network to model the reverse process:

$$
p_{\theta}(x_{t-1}\mid x_{t}) \text{ models unknown target } q(x_{t-1}\mid x_{t})
$$

Wait, why it is _unknown_?

$$
q(x_{t-1}\mid x_{t}) = \frac{q(x_{t-1},x_{t})}{q(x_{_{t}})}=\frac{q(x_{t}\mid x_{t-1})q(x_{t-1})}{q(x_{t})}
$$

When doing reverse, we don't know $q(x_{t-1})$ and $q(x_{t})$. You can imagine the reverse process as we have a starting point $q(x_{T})$, it times some factor $f$ multiple times get $q(x_{0})$, the transition is just like the factor, but we don't know what the real distribution is (or you are the GOD, you know every thing about the world), so we can not solve the equation. Since then, what we know is:

$$
q(x_{t-1}\mid x_{t},x_{0})=\mathcal{N}(x_{t-1}\mid \tilde{\mu}(x_{t},x_{0}),\tilde{\beta}_{t}\mathbf{I})
$$

where:

$$
\tilde{\beta}_{t}\coloneqq \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}}\beta_{t}
$$

$$
\tilde{\mu}_{t}(x_{t},x_{0})\coloneqq \frac{\sqrt{ \bar{\alpha}_{t-1} }\beta_{t}}{1-\bar{\alpha}_{t}}x_{0}+ \frac{\sqrt{ \alpha_{t} }(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_{t}}x_{t}
$$

We also learn from forward process that:

$$
x_{t}=\sqrt{ \bar{\alpha}_{t} }x_{0}+\sqrt{ 1-\bar{\alpha} _{t}}\epsilon
$$

so our mean can be transformed into:

$$
\frac{1}{\sqrt{ \bar{\alpha}_{t} }}\left( x_{t}- \frac{1-\alpha_{t}}{\sqrt{ 1-\bar{\alpha}_{t} }}\epsilon  \right)
$$

_In short_, we are forwarding with a single point but reverse needs the full distribution of real world, which is intractable, so we can only lie to the model let it treat the transition based on specific data point as real transition. With huge amount of data, it can also performs very well.

## Model Noise

How to parameterize $p_{\theta}$ and let it learn $q(x_{t-1}\mid x_{t}, x_{0})$?

We represent $p_{\theta}$ by Gaussian, train by minimizing KL divergence. $D_{\text{KL}}$ of two Gaussian is like L2 loss:

$$
D_{\text{KL}} (\mathcal{N}_{1}\|\mathcal{N}_{2})=\log\left(  \frac{\sigma_{2}}{\sigma_{1}} + \frac{\sigma_{1}^{2}+\|\mu_{1} -\mu_{2}\|^{2}}{2\sigma_{2}^{2}}- \frac{1}{2}  \right)
$$

The true parameterized part of $p_{\theta}$ is its mean, it estimates the _noise_ in $q$, and its var is a preset Gaussian.

**\*Wait what?** We are learning the noise?\* Isn't the noise just Gaussian?

Hold on, hold on. The noise here is no longer the noise we see in forward process, but they are close to each other. Let me dive into it.

Remember we were doing sampling in the forward process right? So we are only doing $T$ times of sampling of Gaussian in the forward process, and we were also doing some linear combination, which means, the noise is different from pure Gaussian! Or we can say, pure Gaussian just never exists for finite operations. Just like we only have limited data set but never have knowledge about infinite world.

So what we are actually doing in forward process is not simply adding Gaussian flavor to input, but adding biased noise to it, which makes its original value annihilate and only Gaussian noise remains. The real noise we applied is biased: consists of Gaussian and negative data point (may not be totally the same, so we need more data).

Since then, the reverse process model totally makes sense! The var part is mainly responsible for the Gaussian part, and our mean, period, is responsible for the data building part. Such separation is already done in $q$, it has $\beta$ part acts as Gaussian.

That's why we are estimating the noise, a trend towards ground truth is buried in it.

## Loss

Alright, come back to divergence. After eliminating useless part, we get final loss:

$$
w_{t}\|\epsilon-\epsilon_{\theta}(x_{t})\|^{2}
$$

$\epsilon_{\theta}(x_{t})$ is a network to predict noise, its input is noisy image. $w_{t}$ represents weights due to $\alpha_{t},\beta_{t}$, but set it as 1 works better empirically.

# Training Objective

Build training objective with variational lower bound (similar to ELBO):

$$
\begin{align*}
\mathcal{L}_{\text{VLB}}&\coloneqq \mathcal{L}_{T} +\mathcal{L}_{T-1}+\dots+\mathcal{L}_{0} \\
\mathcal{L}_{T}&\coloneqq D_{\text{KL}}(q(x_{T}\mid x_{0}) \parallel p_{\theta}(x_{T})) \\
\mathcal{L}_{t-1}&\coloneqq D_{\text{KL}}(q(x_{t-1}\mid x_{0},x_{t}) \parallel p_{\theta}(x_{t-1}\mid x_{t})) \\
\mathcal{L}_{0} &\coloneqq -\log p_{\theta}(x_{0}\mid x_{1})
\end{align*}
$$

Compare to VAE, we don't have parameter for $q$, but also have reconstruction loss in the end and L2 loss on noise.

In the end we have:

$$
\mathcal{L} =\mathbb{E}_{x_{0},t,\epsilon}\left[w_{t} \|\epsilon-\epsilon_{\theta}(x_{t}, t) \|^{2} \right]
$$

$x_{0}$ over $p_{\text{data}}$, $t$ over $[1,T]$, $\epsilon$ over $\mathcal{N}(0,1)$. The separate $t$ in $\epsilon_{\theta}$ is to remind you that it is _conditioned on noise level_.

# Noise Conditional Network

Diffusion models decompose a distribution into **many** simpler ones. We need the same number of networks to fit **all** of them or we can **combine** all into one powerful network, this network is conditioned on noise level $t$.

In the paper studied this, they just pass $x_{t}t$ into $\epsilon_{\theta}$ instead of only $x_{t}$.

# Energy-based Models and Score Matching

This part is kind of bonus......

Diffusion Models are closely related to _Score Matching_. Score Matching is one solution to _Energy-based Models_.

Energy-based Models:

- can be probabilistic or non-probabilistic
- can be generative or discriminative

Many useful concepts in diffusion co-evolved with score matching.

## Energy-based Models

Define a _scalar_ function (output is scalar), called "energy".

At _inference_ time, find $x$ that minimizes energy.

We can use an energy to model a probability distribution:

$$
p(x) = \frac{\exp(-E(x))}{Z}
$$

Then _Score function_ comes out, it is the gradient of log-probability:

$$
\nabla_{x}\log p(x)=-\nabla _{x}E(x)
$$

## Score Matching

Instead of parameterizing $p$, we can parameterize the score, and learn data score with some kind of divergence.

Really similar to Diffusion Model, we can also apply score matching to denoising, use it to match the negative biased noise.

### Langevin Dynamics

Score is gradient, so when we do the "reverse", we sample $x$ from $p$ by iterating:

$$
x_{t}\leftarrow x_{t-1}+ \frac{\sigma^{2}}{2}\nabla_{x}\log p_{\theta}(x_{t-1})+\sigma z_{t}
$$

With $z_{t}\sim \mathcal{N}(0,1)$ then replace score function with energy gradient (don't forget negative).

This also explains why we want to find the $x$ that minimizes energy, because with gradient descent, we are moving forward in reverse until reaching the lowest point which is also the minimal energy.
