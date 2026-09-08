# Probabilistic Latent Variable Models
## Probabilistic Models
Def: models that represents distributions.
Like $p(x)$ or $p(y|x)$, for example: if $y$ is linear with $x$ then we can model $p(y|x)$ by a Gaussian distribution with some noise. And we also have something that is conditional distributed in RL: the policy $\pi_{\theta}(\mathbf{a}|\mathbf{s})$.
## Latent Variable Models
Usually used in mixture distributions.
$p(x)=\sum_{z}p(x|z)p(z)$, $z$ here is the mixture element (latent variable), which decides *categories*.
### Latent Variable Models in General
$p(x)$ is a not simple distribution, $p(z)$ is simple, then $p(x|z)$ is also simple, e.g. $p(x|z)=\mathcal{N}(\mu_{\text{nn}}(z),\sigma_{\text{nn}}(z))$. Since $p(x)=\int p(x|z)p(z)\text{d}z$, we can represent a complex distribution with product and integral of two simple distributions.
### Latent Variable Models in RL
# Variational Inference
## The Variational Approximation
**How to calculate $p(z|x_{i})$?**
Approximate with $q_{i}(z)=\mathcal{N}(\mu_{i},\sigma_{i})$. This can bound $\log p(x_{i})$.
$$
\begin{align}
\log p(x_{i})&=\log \int_{z} p(x_{i}|z)p(z) \\
&=\log \int_{z}p(x_{i}|z)p(z)\frac{q_{i}(z)}{q_{i}(z)} \\
&=\log E_{z\sim q_{i}(z)}\left[\frac{p(x_{i}|z)p(z)}{q_{i}(z)}\right] \\
&\geq E_{z\sim q_{i}(z)}\left[ \log \frac{p(x_{i}|z)p(z)}{q_{i}(z)}\right] \\
&=E_{z\sim q_{i}(z)}[ \log p(x_{i}|z)+\log p(z)]+\mathcal{H}(q_{i}) \\
&=\mathcal{L}_{i}(p,q_{i})
\end{align}
$$
**Jensen's inequality:** $\log E[y]\geq E[\log y]$
A brief aside...
What is Entropy? [[PRML#Entropy]]
What is KL-Divergence? $D_{KL}(q||p)=E_{x\sim q(x)}\left[\log \frac{q(x)}{p(x)}\right]$, it tells how *different* are two distributions.
So we have:
$$
\log p(x_{i})=D_{KL}(q_{i}(x_{i})||p(z|x_{i}))+\mathcal{L}_{i}(p,q_{i})
$$
The lower the KL-divergence is, the tighter our bound is. The problem becomes maximizing $\mathcal{L}_{i}(p,q_{i})$ with respect to $q_{i}$ minimizes KL-divergence!
# Amortized Variational Inference
The problem in Variational Inference is for each $x_{i}$ we have corresponding $q_{i}$ which has 2 parameters $\mu_{i}$ and $\sigma_{i}$, so there would be too many parameters over there. To solve this, we can learn a *network* enables $q_{i}(z)=q(z|x_{i})\approx p(z|x_{i})$.
That is $q_{\phi}(z|x)=\mathcal{N}(\mu_{\phi}(x),\sigma_{\phi}(x))$.
## The Re-parameterize Trick
Now we have:
$$
\begin{align}
\log p(x_{i})&=J(\phi)+\mathcal{H}(q_{\phi}) \\
q_{\phi}(z|x)&=\mathcal{N}(\mu_{\phi}(x),\sigma_{\phi}(x)) \\ \\

\text{Another way to represent distribution of }z&=\mu_{\phi}(x)+\epsilon \sigma_{\phi}(x),\text{ where }\epsilon \sim\mathcal{N}(0,1)\text{ independent of }\phi \\ \\

J(\phi)&=E_{z\sim q_{\phi}(z|x_{i})}[r(x_{i},z)]=E_{z\sim q_{\phi(z|x_{i})}}[\log p(x_{i}|z)+\log p(z)] \\
&=E_{\epsilon \sim\mathcal{N}(0,1)}[r(x_{i},\mu_{\phi}(x_{i})+\epsilon \sigma_{\phi}(x_{i}))]
\end{align}
$$
# Generative Models: Variational Auto-encoders
[[Variational Auto-Encoder]]