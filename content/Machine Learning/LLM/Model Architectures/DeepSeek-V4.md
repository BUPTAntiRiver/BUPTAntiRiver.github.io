# 1. Abstract

DeepSeek-V4 released two new MoE models, both supporting a context length of one million tokens. The key upgrades and optimizations are:

- A hybrid attention architecture that combines Compressed Sparse Attention (CSA) and Heavily Compressed Attention (HCA) to improve long-context efficiency
- [[Manifold-Constrained Hyper-Connections]] to enhance conventional residual connections
- Muon optimizer for faster convergence and greater training stability

# 2. Architecture

V4 retains the Transformer architecture and Multi-Token Prediction (MTP) modules, while introducing several key upgrades over V3 just like mentioned in abstract.

## 2.1. Designs Inherited from DeepSeek-V3

**Mixture-of-Experts.** V4 still adopt the [[DeepSeekMoE]] for feed forward network, but change the activation function that computes the affinity scores from $\text{Sigmoid}(\cdot)$ into $\text{Sqrt}(\text{Softplus}(\cdot))$. Also employ the [[DeepSeek-V3|auxiliary-loss-free]] strategy, augmented by a slight sequence-wise balance loss.

For V4, the constraint on the number of routing target nodes is removed, also the parallelism strategy is redesigned. Furthermore, compared with V3, they replace the dense FFN layers in the initial several Transformer blocks with MoE layers that employ Hash routing.

**Multi-Token Prediction.** Same strategy as V3.

## 2.2. Manifold Constrained Hyper-Connections

Just check my previous blog [[Manifold-Constrained Hyper-Connections]].

## 2.3. Hybrid Attention

Current trend on LLM is agent, which means we need longer and longer context length to deal with larger and more complex task. So 1 million context length is kind of a common suit now, and applying full attention even with super optimized kernel won't be realistic, so we need to have new attention design in order to mitigate the computational bottleneck.
