http://arxiv.org/abs/2401.06066
Conventional MoE architectures like GShard, which activate the top=$K$ out of $N$ experts, face challenges in ensuring expert specialization, which means each expert acquires non-overlapping and focused knowledge. In DeepSeekMoE, they finely segment the experts and isolates $K_{s}$ experts as shared ones, which aiming at capturing common knowledge and mitigating redundancy in routed experts.

# Introduction

Recent research and practices show that with sufficient training data available, **scaling** language models with increased parameters and computation budgets can yield remarkably stronger models. MoE comes as a solution to enable parameter scaling while keeping computational cost at a modest level.
Conventional MoE architectures substitute the FFN in a Transformer with MoE layers. Each MoE layer consists of multiple experts, with each structurally identical to a standard FFN, and each token is assigned to one or two experts through gate. This architecture manifests two potential issues:

1. **Knowledge Hybrid**: existing MoE practices often employ a limited number of experts and thus tokens assigned to a specific expert will be likely to cover diverse knowledge but not specific one.
2. **Knowledge Redundancy**: tokens assigned to different experts may require common knowledge.

In **DeepSeekMoE**, they have two principle strategies:

1. **Fine-Grained Expert Segmentation**: while remaining the number of parameters constant, they segment the experts into a finer grain by splitting the FFN intermediate hidden dimension. This allows diverse knowledge to be decomposed more finely and learned more precisely into different experts. It also increase flexibility in combining knowledge from different experts.
2. **Shared Experts Isolation**: we isolate certain experts to serve as shared experts that are always activated, aiming at capturing common knowledge across varying contexts.
   They first test such architecture on a 2B model, which performs very well, being able to match with 2.9B model. Then they scale up the model parameters to larger models.
   Their main contributions are: **Architectural Innovation**, **Empirical Validation**, **Scalability**, **Alignment for MoE** and **Public Release**.

# Preliminaries: Mixture-of-Experts on Transformers

A standard Transformer block can be represented as:

$$
\begin{align*}
\mathbf{u}^{l}_{1:T} &=\text{Self-Att}(\mathbf{h}^{l-1}_{1:T})+\mathbf{h}^{l-1}_{1:T},\\
\mathbf{h}^{l}_{t}&=\text{FFN}(\mathbf{u}^{l}_{t})+\mathbf{u}^{l}_{t},
\end{align*}
$$

Layer normalization is omitted here.
A typical practice to construct an MoE language model usually substitutes FFNs in a Transformer with MoE layers at specified intervals. If the $l$-th FFN is substituted with an MoE layer, the computation of its output hidden state $\mathbf{h}^{l}_{t}$ is expressed as:

$$
\begin{align}
\mathbf{h}^{l}_{t} &= \sum ^{N}_{i=1}(g_{i,t}\text{FFN}(\mathbf{u}^{l}_{t})) + \mathbf{u}^{l}_{t}, \\
g_{i,t}&=
\begin{cases}
s_{i,t},& s_{i,t}\in \text{Topk}(\{s_{j,t}|1\leq j\leq N\},K), \\
0,& \text{otherwise},
\end{cases} \\
s_{i,t}&=\text{Softmax}_{i}({\mathbf{u}^{l}_{t}}^{T}\mathbf{e}^{l}_{i}),
\end{align}
$$

$g_{i,t}$ denotes the gate value for the $i$-th expert, $s_{i,t}$ denotes the token-to-expert affinity and $\mathbf{e}^{l}_{i}$ is the centroid of the $i$-th expert in the $l$-th layer. Note that $g_{i,t}$ is sparse, and this sparsity ensures computational efficiency with an MoE layer.

# DeepSeekMoE Architecture

2 principle strategies: fine-grained expert segmentation and shared expert isolation.

## Fine-Grained Expert Segmentation

They want each token to be routed to more experts, so diverse knowledge will gain the potential to be decomposed and learned in different experts respectively.
They segment each expert FFN into $m$ smaller experts by reducing the FFN intermediate hidden dimension to $\frac{1}{m}$ times its original size. Since each expert becomes smaller, in response they also increase the number of activated experts to $m$ times to keep the same computation cost. The parameters amounts are the same with standard MoE but the potential combinations are greatly increased.

## Shared Expert Isolation

With a conventional routing strategy, tokens assigned to different experts might have some necessary common knowledge, and this leads to redundancy for duplicate common sense in multiple experts. So they isolate $K_{s}$ shared experts to capture and consolidating common knowledge across varying contexts. Finally the total number of routed experts are $mN-K_{s}$ to keep computation cost the same.

## Load Balance Consideration

To avoid load imbalance, which have 2 notable defect:

1. the risk of routing collapse, only a few experts are always visited and other experts are not well trained.
2. if experts are distributed across multiple devices, load imbalance can exacerbate computation bottlenecks.
   **Expert-Level Balance Loss**
   $$
   \begin{align}
   \mathcal{L}_{\text{ExpBal}} &=\alpha_{1}\sum ^{N'}_{i=1}f_{i}P_{i}, \\
   f_{i} &= \frac{N'}{K'T}\sum ^{T}_{t=1} \mathbb{1}(\text{Token }t\text{ selects Expert }i), \\
   P_{i} &= \frac{1}{T}\sum ^{T}_{t=1}s_{i,t},
   \end{align}
   $$
   where $N'=mN-K_{s},K'=mK-K_{s}$ for brevity.
   **Device-Level Balance Loss**. If we partition all experts into $D$ groups $\{ \mathcal{E}_1,\mathcal{E}_2,\ldots,\mathcal{E}_D \}$, each group in on one device, the loss can be computed as follows:
   $$
   \begin{align*}
   \mathcal{L}_{\mathrm{DevBal}} &= \alpha_2 \sum_{i=1}^{D} f'_i P'_i, \\
   f'_i &= \frac{1}{|\mathcal{E}_i|} \sum_{j \in \mathcal{E}_i} f_j, \\
   P'_i &= \sum_{j \in \mathcal{E}_i} P_j.
   \end{align*}
   $$
   where $\alpha_{1},\alpha_{2}$ are both hyper-parameters. In practice, they set a _small_ expert-level balance factor to mitigate the risk of routing collapse and meanwhile set a _larger_ device-level balance factor to promote balanced computation across devices. When aiming to alleviate computation bottleneck, then it becomes _unnecessary_ to enforce strict balance constraints at the expert level, because excessive constraints on load balance will compromise model performance.
