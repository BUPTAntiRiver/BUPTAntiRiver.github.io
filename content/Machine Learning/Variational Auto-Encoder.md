We are calling it an auto-encoder but it is actually made up of two parts: an encoder and a decoder.

But before step into the notion of variational auto-encoder, we may check the auto-encoder without the adjective.

# Auto-Encoder

Take MNIST for example, we have input image $x$ and **encoder** is responsible for mapping $x$ to a lower dimension latent data $z$, like the number it represents or the label. This is also called the **inference** part, we get some latent information from the raw input data. In contrast, we have **decoder** which can reconstruct the the number image from a given latent input. This is also called **generate** part.

So a auto-encoder first use a neural network as encoder to transform the input into a intermediate value in the latent space, then use another neural network as decoder to generate a image back, then we evaluate the difference between reconstructed image and original input as loss.

After training, when we give the decoder a random input, it may generate something comes from MNIST.

But wait... Doesn't this seem to be too arbitrary? Can such simple architecture do other complex work? Of course it is not good enough, that is why we have Variational Auto-Encoder.

# Variational Auto-Encoder

## Idea

The difference between variational auto-encoder and auto-encoder is... variational! But what is variational? It actually means we want to add restrictions to the latent space the encoder can encode the input to, for example if we model the latent variable $z$ with a simple Gaussian, then the output of encoder will be mean and standard variance. And our decoder will sample from this distribution to generate output. This action improves the model's generalization ability, it process with data beyond the dataset better.

It also expands the exploring space, we can design different prior (latent variable) setting for different tasks, optimize over the prior and so on.

## Loss

Measuring how good a distribution is, we can use KL-divergence:

$$
\min_{\theta}\mathcal{D}(p_{\text{data}}\,\|\,p_{\theta})
$$

which equals to maximize likelihood:

$$
\max_{\theta}\mathbb{E}_{x\sim p_{\text{data}}}\log p_{\theta}(x)
$$

with $p_{\theta}(x)$ represented as:

$$
p_{\theta}(x)=\int_{z}p_{\theta}(x\mid z)p(z)dz
$$

But raw $p(z)$ is intractable, so we propose a $q_{\phi}(z|x)$ which is tractable, trainable, and we use $q(z)$ for simplicity.

$$
\begin{align}
\log p_\theta(x)
&= \int_z q(z)\,\log p_\theta(x)\,dz \\[6pt]
&= \int_z q(z)\,\log\!\left(\frac{p_\theta(x\mid z)p_\theta(z)}{p_\theta(z\mid x)}\right)dz \\[6pt]
&= \int_z q(z)\,\log\!\left(\frac{p_\theta(x\mid z)p_\theta(z)}{p_\theta(z\mid x)}\frac{q(z)}{q(z)}\right)dz \\[6pt]
&= \int_z q(z)\left(
\log p_\theta(x\mid z)
+ \log\frac{p_\theta(z)}{q(z)}
+ \log\frac{q(z)}{p_\theta(z\mid x)}
\right)dz \\[6pt]
&= \mathbb{E}_{z\sim q(z)}[\log p_\theta(x\mid z)]
- D_{\mathrm{KL}}(q(z)\,\|\,p_\theta(z))
+ D_{\mathrm{KL}}(q(z)\,\|\,p_\theta(z\mid x))
\end{align}
$$

The first two terms in the last line are tractable and the last term is intractable, so we move it to the other side can let the left two terms be the evidence lower bound of the $\log p_{\theta}(x)$.

To maximize evidence lower bound, we need to minimize the negate of right hand side (treat it like a loss), and the first term is called **reconstruction loss** which tells how $x'$ similar to $x$, the second term is called **regularization loss** which tells how $q_{\phi}(z|x)$ similar to $p(z)$ (our predefined prior) so that it won't be too arbitrary and lose generalization ability.

### Reconstruction Loss

$$
-\mathbb{E}_{z\sim q(z\mid x)}[\log p_{\theta}(x\mid z)]
$$

The inner part tells the probability we get the input $x$ given the latent variable $z$. So the bigger this term, the lower the loss. We can also dive into particular cases. Suppose we have a Gaussian model decoder, then the inner part will be like:

$$
\frac{1}{2\sigma^{2}_{0}}\|x-x'\|+\text{const}
$$

This is exactly L2 loss, if the model is Bernoulli, we will have binary reconstruction cross-entropy loss.

# Additional Materials

[Introducing Variational Autoencoders (in Prose and Code)](https://blog.fastforwardlabs.com/2016/08/12/introducing-variational-autoencoders-in-prose-and-code.html)
