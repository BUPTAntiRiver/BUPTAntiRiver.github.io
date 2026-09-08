So far, reward function is manually designed to define a task. Now I want to *learn* the reward function from observing an expert.
# Learning the Reward Function
## Learning the optimality variable
$p(\mathcal{O}_{t}|\mathbf{s}_{t},\mathbf{a}_{t},\psi)=\exp(r_{\psi}(\mathbf{s}_{t},\mathbf{a}_{t}))$
given: samples $\{\tau_{i}\}$ sampled from $\pi^\star(\tau)$
$$p(\tau|\mathcal{O}_{1:T},\psi)\propto p(\tau)\exp\left(\sum_{t}r_{\psi}(\mathbf{s}_{t},\mathbf{a}_{t})\right)$$
can ignore $p(\tau)$ because it is independent of $\psi$.
Maximum Likelihood Learning:
$$
\max_{\psi} \frac{1}{N}\sum_{i=1}^N\log p(\tau_{i}|\mathcal{O}_{1:T},\psi)=\max_{\psi} \frac{1}{N}\sum_{i=1}^Nr_{\psi}(\tau_{i})-\log Z
$$
$Z$ is the normalizer, which prevents the model just assign huge values to any trajectories.
## The IRL partition function
The final form of Maximum Likelihood Learning is called IRL partition function.
$$
\max_{\psi} \frac{1}{N}\sum_{i=1}^Nr_{\psi}(\tau_{i})-\log Z
$$
$Z$ is the hard part to deal with.
$$
Z=\int p(\tau)\exp(r_{\psi}(\tau))\text{d}\tau
$$
### Max Entropy IRL
# IRL and GANs
It looks like a game...
## Generative Adversarial Networks
[[GAN]]