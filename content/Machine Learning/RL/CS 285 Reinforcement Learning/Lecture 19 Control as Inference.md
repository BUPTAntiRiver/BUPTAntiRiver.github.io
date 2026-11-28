In this lecture I'm going to talk about how to re-frame control problems as inference problems.
**Goals**:
- Understand the connection between inference and control
- Understand how specific RL algorithms can be instantiated in this framework
- Understand why this might be a good idea
# Optimal Control as a Model of Human Behavior

# Control as Variational Inference
## Optimism Problem
$$
\begin{align*}
& \text{for } t = T-1 \text{ to } 1 \text{:} \\
& \qquad\beta_t(\mathbf{s}_t, \mathbf{a}_t) = p(\mathcal{O}_t|\mathbf{s}_t, \mathbf{a}_t)E_{\mathbf{s}_{t+1} \sim p(\mathbf{s}_{t+1}|\mathbf{s}_t, \mathbf{a}_t)}[\beta_{t+1}(\mathbf{s}_{t+1})] \\
& \qquad\beta_t(\mathbf{s}_t) = E_{\mathbf{a}_t \sim p(\mathbf{a}_t|\mathbf{s}_t)}[\beta_t(\mathbf{s}_t, \mathbf{a}_t)] \\
& \text{let } V_t(\mathbf{s}_t) = \log \beta_t(\mathbf{s}_t) \\
& \text{let } Q_t(\mathbf{s}_t, \mathbf{a}_t) = \log \beta_t(\mathbf{s}_t, \mathbf{a}_t)
\end{align*}
$$
With marginalizing and conditioning, we get: $p(\mathbf{a}_{t}|\mathbf{s}_{t},\mathcal{O}_{1:T})$ (the policy, which we want) and $p(\mathbf{s}_{t+1}|\mathbf{s}_{t},\mathbf{a}_{t},\mathcal{O}_{1:T})\neq p(\mathbf{s}_{t+1}|\mathbf{s}_{t},\mathbf{a}_{t})$ (the transition probability, we don't want)
So now I want to know, given that I obtained high reward, what was my action probability, *given that my transition probability did not change*?
I need to find another distribution $q(\mathbf{s}_{1:T},\mathbf{a}_{1:T})$ that is close to $p(\mathbf{s}_{1:T},\mathbf{a_{1:T}}|\mathcal{O}_{1:T})$ but has dynamics $p(\mathbf{s}_{t+1}|\mathbf{s}_{t},\mathbf{a}_{t})$.
This is similar to variational inference!
$$
V_{t}(\mathbf{s}_{t})=\log \int \exp(Q_{t}(\mathbf{s}_{t},\mathbf{a}_{t}))\text{d}\mathbf{a}_{t}
$$
This is called soft maximum.
$$
Q_{t}(\mathbf{s}_{t},\mathbf{a}_{t})=r(\mathbf{s}_{t},\mathbf{a}_{t})+E[V_{t+1}(\mathbf{s}_{t+1})]
$$
# Algorithms for RL as Inference
