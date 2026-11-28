# Metrics used to evaluate RL methods
## Sample Complexity
How many transitions/episodes do I need to obtain a good policy?
$$N=\mathcal{O}\left(\text{poly}\left(|S|,|A|,\dfrac{1}{1-\gamma}\right)\right)\quad \text{then} \ \max_{s,a}|Q^\pi (s,a)-\hat{Q}^\pi (s,a)|\leq \epsilon$$
## Regret
Measures how fast the policy converges to the optimal policy.
$\pi_{0}, \pi_{1},\pi_{2},\dots,\pi_{N}$
$$
\begin{flalign*}
\text{Reg}(N)=\sum^N_{i=1}E_{s_{0}\sim \rho}[V^*(s_{0})]-E_{s_{0}\sim\rho}[V^{\pi_{i}}(s_{0})] &&
\end{flalign*}
$$
$\text{Reg}(N)=\mathcal{O}(\sqrt{ N })$
# Assumptions used in RL Analysis
We can break down the RL into 2 parts:
- the exploration part
- given data from the exploration policy, we should be able to learn from it
## Preliminaries
**Concentration**:
*Hoeffding's inequality*
Suppose $X_{1,}, X_{2}, \dots,X_{n}$ are a sequence of independent, identically distributed (i.i.d.) random variables with mean $\mu$. Let $\bar{X}=n^{-1}\sum ^{n}_{i=1}X_{i}$. Suppose that $X_{i}\in [b_{-},b_{+}]$ with probability $1$, then
$$
\begin{align}
P(\bar{X}\geq \mu+\epsilon)\leq e^{ -2n\epsilon^{2}/(b_{+}-b_{-})^{2}}. \\
P(\bar{X}\geq \mu-\epsilon)\leq e^{ -2n\epsilon^{2}/(b_{+}-b_{-})^{2}}. 
\end{align}
$$
$$
\bar{X}_{n} - E[X]\leq (b_{+}-b_{-})\sqrt{ \frac{\ln\left( \frac{1}{\delta} \right)}{2n} }
$$
The above equation says that average over samples gets closer to the mean.

More complex variants:
*Concentration for Discrete Distributions*
Let $z$ be a discrete random variable that takes values in $\{1, \dots, d\}$, distributed according to $q$. We write $q$ as a vector where $\vec{q} = [\Pr(z=j)]_{j=1}^d$. Assume we have $N$ i.i.d. samples, and that our empirical estimate of $\vec{q}$ is $[\hat{q}]_j = \sum_{i=1}^N 1[z_i=j]/N$.
We have that $\forall \epsilon > 0$:
$$
\Pr \left(||\hat{q}-\vec{q}||_{2}\geq \frac{1}{\sqrt{ N }}+\epsilon\right)\leq e^{ -N\epsilon^{2} }.
$$
# Model-free RL
## Analyzing fitted Q-iteration
abstract model of exact Q-iteration: $Q_{k+1}\leftarrow TQ_{k}$
$$TQ=r + \gamma P\max_{a}Q,T \text{ is Bellman operator}$$
abstract model of approximate fitted Q-iteration: 
$$
\hat{Q}_{k+1}\leftarrow \arg\min_{\hat{Q}}\lvert \lvert \hat{Q}-\hat{T}\hat{Q}_{k} \rvert  \rvert \text{ which norm to use?}
$$
The $\hat{T}$ here is approximate Bellman operator: 
$$
\begin{align}
\hat{T}Q&=\hat{r}+\gamma \hat{P}\max_{a}Q \\
\hat{r}&=\frac{1}{N(s,a)}\sum_{i}\delta((s_{i},a_{i})=(s,a))r_{i} \\
\hat{P}(s'|s,a)&=\frac{N(s,a,s')}{N(s,a)}
\end{align}
$$
**Note:** these are not models, this is the effect of averaging together transitions in the data.
Errors come from $\hat{T}$ and $\hat{T}\hat{Q}_{k}$, sampling error and approximation error.