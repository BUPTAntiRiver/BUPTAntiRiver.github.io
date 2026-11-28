# Recap: Offline Reinforcement Learning
## on-policy RL
Run policy to get new data and discard old data.
## off-policy RL
A buffered version of on-policy RL, all the data collected would be stored in a buffer.
## offline reinforcement learning
No data collecting, in offline RL, we have a data set collected by some unknown behavior policy.
**Formally**:
$\mathcal{D} = \{(\mathbf{s}_{i},\mathbf{a}_{i},\mathbf{s}_{i}',r_{i})\}$
$\mathbf{s}\sim d^{\pi_{\beta}}(\mathbf{s})$
$\mathbf{a}\sim \pi_{\beta}(\mathbf{a}|\mathbf{s})$ generally **not** known
$\mathbf{s}'\sim p(\mathbf{s}'|\mathbf{s},\mathbf{a})$
$r\leftarrow r(\mathbf{s},\mathbf{a})$
RL object is just like before, which is to maximize expected reward.

