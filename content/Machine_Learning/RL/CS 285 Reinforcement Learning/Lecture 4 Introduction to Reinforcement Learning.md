# Definitions
## Terminology and notation
[[Machine Learning/RL/CS 285 Reinforcement Learning/Lecture 2]]
**Partially observed Markov Decision Process**
$\mathcal{M}=\{\mathcal{S,A,O,T,E,r}\}$
$$
\begin{align*}
\mathcal{S}&-\text{state space}&\text{states }s\in\mathcal{S}\text{ (discrete or continuous)}\\
\mathcal{A}&-\text{action space}&\text{actions }a\in\mathcal{A}\text{ (discrete or continuous)}\\
\mathcal{O}&-\text{observation space}&\text{observations }o\in\mathcal{O}\text{ (discrete or continuous)}&\\
\mathcal{T}&-\text{transition operator (a tensor)}\\
\mathcal{E}&-\text{emission probability } p(o_t|s_t)\\
r&-\text{reward function}&r:\mathcal{S}\times\mathcal{A}\rightarrow\mathbb{R}
\end{align*}
$$
## The goal of reinforcement learning
$$p_\theta(\tau)=p_\theta(\textbf{s}_1,\textbf{a}_1,\ldots,\textbf{s}_T,\textbf{a}_T)=p(\textbf{s}_1)\prod^T_{t=1}\pi_\theta(\textbf{a}_t|\textbf{s}_t)p(\textbf{s}_{t+1}|\textbf{s}_t,\textbf{a}_t)$$
$$\theta^\star=\arg\max_\theta E_{\tau\sim p_\theta(\tau)}\left[\sum_tr(\mathbf{s}_t,\mathbf{a}_{t})\right]$$
The $\tau$ here means trajectory. So the best $\theta$ here maximize the expectation of rewards according to the distribution of $\tau$.
We can combine $\mathbf{a}_{t},\mathbf{s}_{t}$ to build a new Markov Chain about $(\mathbf{s}_{t},\mathbf{a}_{t})$:
$$p((\mathbf{s}_{t+1},\mathbf{a}_{t+1})|(\mathbf{s}_{t},\mathbf{a}_{t}))=p(\mathbf{s}_{t+1}|\mathbf{s}_t,\mathbf{a}_t)\pi_{\theta}(\mathbf{a}_{t+1}|\mathbf{s}_{t+1})$$
The length of $\tau$ is the horizon of our model, which means how many steps do we take into consideration.
### Finite horizon case: state-action marginal
$$
\begin{align}
\theta^\star&=\arg\max_\theta E_{\tau\sim p_\theta(\tau)}\left[\sum_tr(\mathbf{s}_t,\mathbf{a}_{t})\right]\\
&=\arg\max_{\theta}\sum^T_{t=1}E_{(\mathbf{s}_{t},\mathbf{a}_{t})\sim p_{\theta}(\mathbf{s}_{t},\mathbf{a}_{t})}[r(\mathbf{s}_{t},\mathbf{a}_{t})]
\end{align}
$$
The equation here calculates the expectation of reward step by step.
### Infinite horizon case: stationary distribution
What if $T =\infty$?
$p(\mathbf{s}_{t},\mathbf{a}_{t})$ converges to a *stationary distribution*, we divide the summation of expectation by $T$, which does not change the result of $\theta$ but makes the right hand side converge **in the limit** as $T\rightarrow \infty$.
$$\theta^\star=\arg\max_{\theta}E_{(\mathbf{s},\mathbf{a})\sim p_{\theta}(\mathbf{s},\mathbf{a})}[r(\mathbf{s},\mathbf{a})]$$
### Summary
Two cases have been shown, and this is why we can apply gradient descent even on discrete reward functions, because we are maximizing the **expectation** of rewards.
# Algorithms
## Why so many $\text{RL}$ algorithms?
### Different trade-offs
#### Sample efficiency
Off policy: able to improve the policy without generating new samples from that policy
On policy: each time the policy is changed, we need to generate new samples
#### Stability & ease of use
Does the model converge?
If it converges, to what? A local minimum or a global one?
And does it converge every time?
These questions emerge because $\text{RL}$ is unlike supervised learning, it not always uses gradient descent.
# Value Functions
## Q-function
$Q^\pi =\sum^T_{t'=t}E_{\pi_{\theta}}[r(\mathbf{s}_{t'},\mathbf{a}_{t'})|\mathbf{s}_{t},\mathbf{a}_{t}]\text{: total reward from taking }\mathbf{a}_{t} \text{ in }\mathbf{s}_{t}$
## Value function
$V^\pi (\mathbf{s}_{t})=\sum^T_{t'=t}E_{\pi_{\theta}}[r(\mathbf{s}_{t'},\mathbf{a}_{t'})|\mathbf{s}_{t}]\text{: total reward from   }\mathbf{s}_{t}$
$V^\pi (\mathbf{s}_{t})=E_{\mathbf{a}_{t}\sim \pi(\mathbf{a}_{t}|\mathbf{s}_{t})}[Q^\pi (\mathbf{s}_{t},\mathbf{a}_{t})]$
$E_{\mathbf{s}_{1}\sim p(\mathbf{s}_{1})}[V^\pi (\mathbf{s}_{1})]\text{ is the RL objective!}$
