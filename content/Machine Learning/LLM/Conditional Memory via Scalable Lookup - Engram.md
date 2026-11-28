Paper Link: http://arxiv.org/abs/2601.07372

# Introduction

Modern large language models scales capacity via conditional computations like [[MoE-Mixture-of-Experts]], which increases model size without proportional increases in compute.

Despite the success of this conditional computation paradigm, the intrinsic heterogeneity of linguistic signals suggests significant room for _structural optimization_. Specifically, language modeling entails two main different sub-tasks: **compositional reasoning** and **knowledge retrieval**.

The former task needs deep dynamic computation and text about name entities and formulaic patterns, which is local, static and highly stereotyped. Such information can be represented as computationally inexpensive lookups with classical $N$-gram models. However, current LLMs are forced to **simulate retrieval through computation** (query, key, value, [[Transformers and LLM]]).

To align model with this linguistic duality, we advocate for a complementary axis of sparsity: **conditional memory**. Just like conditional computation sparsely activates parameters to process dynamic logic, conditional memory relies on sparse lookup operations to retrieve static embeddings for fixed knowledge. They revisited $N$-gram embeddings and propose a novel module **Engram**, a conditional memory module grounded in the classic $N$-gram structure but equipped with **modern adaptations** such as tokenizer compression, multi-head hashing, contextualized gating, and multi-branch integration.

# Architecture

## Overview

Engram is a conditional memory module designed to separate static pattern storage from dynamic computation in order to enhance Transformer backbone.

## Sparse Retrieval via Hashed $N$-grams

The first phase maps local context to entries, then retrieves embeddings.

### Tokenizer Compression

Modern tokenizers has different id for tokens like: `apple` and `␣apple`, in order to maximize semantic density, we project them to same id based on normalized textual equivalence (using NFKC, lowercasing, etc.). For raw id $x_{t}$ we have $x'_{t}$ then have $N$-gram $g_{t,n}=(x'_{t-n+1},\dots,x'_{t})$.

### Multi-Head Hashing

It is intractable to parametrize all possible $N$-grams. We adopt a hashing-based approach. We have $K$ distinct hash functions for each $N$-gram order $n$. Each head $k$ maps the compressed context to an index within an embedding table $\mathbf{E}_{n,k}$ of prime size $M_{n,k}$ via a deterministic function $\varphi_{n,k}$:

$$
z_{t,n,k}\triangleq\varphi_{n,k}(g_{t,n}),\quad \mathbf{e}_{t,n,k}=\mathbf{E}_{n,k}[z_{t,n,k}]
$$

In practice, $\varphi$ is implemented as a lightweight multiplicative-XOR hash. We construct the final memory vector $\mathbf{e}_{t}\in \mathbb{R}^{d_{\text{mem}}}$ by concatenating all retrieved embeddings.

## Context Aware Gating

The raw embedding we get knows nothing about context. To enhance expressivity, we apply an Attention-like gating mechanism. We use current hidden state $\mathbf{h}_{t}$ as query, and project embedding to get key and value:

$$
\mathbf{k}_{t}=\mathbf{W}_{K}\mathbf{e}_{t},\quad \mathbf{v}_{t}=\mathbf{W}_{V}\mathbf{e}_{t}
$$

then apply pre-[[RMSNorm]] and compute to get the scalar gate $\alpha_{t}\in(0,1)$:

$$
\alpha_{t}=\sigma\left(  \frac{\text{RMSNorm}(\mathbf{h}_{t})^{\top}\text{RMSNorm}(\mathbf{k}_{t})}{\sqrt{ d }}  \right)
$$

The gated output is defined as $\tilde{\mathbf{v}}_{t}=\alpha_{t}\cdot \mathbf{v}_{t}$.

Finally, to expand the receptive field and enhance the model's non-linearity, we introduce a convolution with kernel size $w$ of 4, dilation $\delta$ of the max $N$-gram order and SiLU activation for sequence of gated values $\tilde{\mathbf{V}}\in \mathbb{R}^{T\times d}$:

$$
Y=\text{SiLU}(\text{Conv1D}(\text{RMSNorm}(\tilde{\mathbf{V}})))+\tilde{\mathbf{V}}
$$

Then Engram is integrated into the _backbone_ via a residual connection: $\mathbf{H}^{(l)}=\mathbf{H}^{(l)}+\mathbf{Y}$, then followed by standard Attention and MoE. Engram is not applied to every layer; its specific placement is governed by the system-level latency constraints detailed in later sections.

## Integration with Multi-branch Architecture

We adopt the advanced multi-branch architecture as our default _backbone_. A defining characteristic of this architecture is the expansion of the residual stream into $M$ parallel branches, where information flow is modulated by learnable connection weights.

We implement a parameter-sharing strategy: a single sparse embedding table and a Value projection matrix $\mathbf{W}_{V}$ are shared across all $M$ branches, whereas $M$ distinct Key projection matrices $\{\mathbf{W}_{k}^{(m)}\}^{M}_{m=1}$ are employed to enable branch specific gating behaviors. The gate signal is identity to Engram gating:

$$
\alpha_{t}^{(m)}=\sigma\left(  \frac{\text{RMSNorm}(\mathbf{h}_{t}^{(m)})^{\top}\text{RMSNorm}(\mathbf{W}_{K}^{(m)}\mathbf{e}_{t})}{\sqrt{ d }}  \right)
$$

The retrieved memory is then modulated by these independent gates applied to the shared value vector: $\mathbf{u}^{(m)}_{t}=\alpha_{t}^{(m)}\cdot(\mathbf{W}_{V}\mathbf{e}_{t})$. Unless otherwise stated, all experiments utilize this integration with [[Manifold-Constrained Hyper-Connections]] ($M=4$).

## System Efficiency: Decoupling Computed and Memory

Unlike MoE, which relies on runtime hidden states for dynamic routing, Engram's retrieval indices depend solely on the input token sequence. So it can have specialized optimization during training and inference.

![[System implementation of Engram.png]]

During _training_, to accommodate large-scale embedding tables, we employ _standard model parallelism_ by sharding the tables across available GPUs. An All-to-All communication primitive is used to gather active rows in the forward pass and dispatch gradients in the backward pass.

During _inference_, this deterministic nature enables a prefetch-and-overlap strategy. Since memory indices are known prior to the forward pass, the system can asynchronously retrieve embeddings from abundant host memory via PCIe. So we should place Engram layer carefully, so that we can take the layers before it as a buffer of **latency masking**, but from experiments, it shows that the **earlier** we have Engram intervention to offload local pattern reconstructions, the better modeling performance we can have. It becomes a trade-off, therefore, the optimal placement must **simultaneously satisfy both modeling and system latency constraints.** In short, if we put Engram early, we don't have enough latency masking, so that GPU may stall, but has better performance due to early reconstruction of local patterns, and if we put Engram deeper, it's just the opposite.

DeepSeek guys are insane, they considered that natural language $N$-grams inherently follow a Zipfian distribution, where a small fraction of patterns accounts for the vast majority of memory access. This statistical property motivates a Multi-Level Cache Hierarchy.

# Scaling Laws and Sparsity Allocation

Two key questions drive the research:

1. **Allocation under Finite Constraints.** When total parameters and training compute are fixed (Iso-parameters and Iso-FLOPs), how should we split the sparse capacity between MoE experts and Engram embeddings?
2. **Infinite Memory Regime.** Considering the non-scaling $\mathcal{O}(1)$ overhead of Engram, if the memory budget is relaxed or scaled aggressively, what scaling behavior does Engram exhibit by itself?

## Optimal Allocation Ratio Between MoE and Engram

**Compute-match Formulation.** We analyze the trade-off using three parameter metrics:

- $P_{\text{tot}}$: total trainable parameters, excluding vocabulary embedding and LM head.
- $P_{\text{act}}$: activated parameters per token. This quantity determines the training cost (FLOPs).
- $P_{\text{sparse}}\triangleq P_{\text{tot}}-P_{\text{act}}$: the _inactive_ parameters, which represent the "free" parameter budget available for scaling model size without incurring computational cost (e.g. unselected experts or unretrieved embeddings).

We keep $P_{\text{tot}}$ and $P_{\text{act}}$ fixed within each FLOPs budget, so that the model have same number of parameters and same per-token FLOPs. For MoE, $P_{\text{act}}$ is determined by the top-$k$ selected experts. For Engram, only a constant number of slots are retrieved per token, so scaling the number of embedding slots increases $P_{\text{tot}}$ without increasing per-token FLOPs.

**Allocation ratio.** We define the allocation ratio $\rho\in[0,1]$ as the fraction of the inactive-parameter budget assigned to MoE expert capacity:

$$
P_{\text{MoE}}^{(\text{sparse})}=\rho P_{\text{sparse}},\quad P_{\text{Engram}}=(1-\rho)P_{\text{sparse}}.
$$

Intuitively:

- $\rho=1$ corresponds to a pure MoE model.
- $\rho<1$ reduces the number of routed experts and reallocates the freed parameters to Engram embedding slots.

Then we get experiment results (left one):

![[Sparsity allocation and Engram scaling.png]]

it's a U-like curve (I really doubt that!), which confirms the two modules are structurally complementary to each other:

- **MoE-dominated:** forcing into inefficient reconstruct.
- **Engram-dominated:** model loses conditional computation capacity, hurting tasks that require dynamic context dependent reasoning.

## Engram under Infinite Memory Regime

Now we fix a MoE backbone and attach an Engram table to it. We sweep the number of slots $M$ to get scaling results, comparing to OverEncoding as baseline.

The result shows in the right part of above figure, which demonstrates that scaling the number of memory slots yields a clear and consistent improvement of validation loss. The curve follows a strict **power law** (linear in log-space).

# Large Scale Pre-training

In this part, they scale up the model. From the experiments result, we can see that Engram really works, increases performance in all aspects comparing to pure MoE.

Also, they trained a model with only more Engram size but same MoE size on the same budget, the result shows that the bigger model has lower loss and better performance on many of the tasks, which suggests that the expanded memory capacity has not yet fully saturated within the current token budget.

# Long Context Training

In this part, they want to research how does Engram affects model's performance on long context tasks.

Results shows, the Engram model with same loss as MoE performs better on long context benchmarks. And which same budget, Engram is no doubt even better.

# Analysis

## How does Engram work?

Engram is actually adding more structural handling to token directly. If we think of this carefully, we find that it has been a long time since last time researches puts effort in making the model makes more sense (I know there are works do such kind of thing, but mainly focuses on scale stuffs). This is just like CNN, with convolution, it can build information from a region, and RNN, can build information from time sequence, and Transformer which utilizes Attention, which can grab information from long time ago easily, so even better context. Other models like diffusion models, flow models, they all have their philosophy.

And Engram is kind of making the model closer to human brain. Brain has many districts in it, each responsible for different jobs, but nowadays model seems to just have a block and keep staking it, making every thing a "mess". Engram is kind of extracting different parts of the brain, so it achieves better performance.

In their experiments, they show that with Engram the understanding to tokens at layer 5 is already similar to pure MoE at layer 12. So it does help the model to reconstruct information better. Check the paper to know about their methodology.

## Ablations

Remember we have done many other augmentations to Engram like multi-branching, context-aware gating, tokenier compression. They do ablations and find any of them is very important. Remove any one can cause great regression in validation loss. However the lightweight depthwise convolution does not matter a lot.

## System Efficiency

Very good. A little overhead.

# Conclusion

Introduces **Conditional Memory** Engram, as an additional sparsity axis aiming to resolve the inefficiency of simulating knowledge retrieval through dynamic computation. It effectively "deepen" the network by relieving early layers from static reconstruction tasks, thereby freeing up attention capacity to focus on global context and complex reasoning.
