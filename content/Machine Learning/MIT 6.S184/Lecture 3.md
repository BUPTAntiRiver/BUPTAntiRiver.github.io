## 4.1. Flow matching

**Goal**: $u_{t}^\theta\sim u_{t}^\text{target}$
**Flow matching loss**:

$$
\mathcal{L}_{\text{fm}}(\theta)=\mathbb{E}\left[\lvert \lvert u_{t}^\theta(x)-u_{t}^\text{target}(x) \rvert  \rvert^2  \right]
$$

**Conditional Flow matching loss**:

$$
\mathcal{L}_{\text{cfm}}(\theta)=\mathbb{E}\left[||u_{t}^\theta(x)-u_{t}^\text{target}(x|z)||^2\right]
$$

$$
\begin{align}
t&\sim \text{uniform distribition in }[0,1] \\
z&\sim p_{\text{data}} \\
x&\sim p_{t}(\cdot|z)
\end{align}
$$

**Theorem 4.1.**:

$$
\mathcal{L}_{\text{fm}}(\theta)=\mathcal{L}_{\text{cfm}}(\theta)+C
$$

for $C>0$ is independent of $\theta$.
