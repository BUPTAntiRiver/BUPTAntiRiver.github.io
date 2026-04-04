---
aliases:
  - STEM
---

Paper link: http://arxiv.org/abs/2601.10639

This paper is said to explore architecture in large language models even earlier than [[Conditional Memory via Scalable Lookup - Engram|DeepSeek Engram]]. Also I wants to make use of the paper reading method I have learned from a paper.

# First Pass

This paper is mainly about how to improve current system performance on _popular_ sparse models. Sparse models means, though the parameter capacity or you can say model size is very big, but the activated parameters during computation are only part of them, which could be rather small, like [[MoE-Mixture-of-Experts|MoE]]. This method is _popular_ is because it can increase model size but not increase computation cost.

The problem of current sparse models is that, sparse computation might not fully utilize compute power, lack parameter access locality, degrade kernel efficiency and raise the number of all-to-all message communication, yielding suboptimal end-to-end performance.

So, STEM does not have such kind of problems, it reduces active parameter number, offload communication latency to CPU side and has better performance.

# Second Pass

## Background

Consider a decoder only transformer with $N$ layers, vocabulary size $V$, model width $d$, and feed-forward width $d_{\text{ff}}$. For a given layer $l$, the SwiGLU feed-forward network block uses a gate projection $\mathbf{W}^{g}_{l}\in \mathbb{R}^{d_{\text{ff}}\times d}$, an up projection $\mathbf{W}^{u}_{l}\in \mathbb{R}^{d_{\text{ff}}\times d}$, and a down projection $\mathbf{W}^{d}_{l}\in \mathbb{R}^{d\times d_{\text{ff}}}$. Consider $t\in\{1,\dots,V\}$ denote the vocabulary id of the current token, and the corresponding input hidden state of the $l^{\text{th}}$ FFN layer is given by $\mathbf{x}_{l}\in \mathbb{R}^{d}$. Then the transformation in the FFN layer is

$$
\mathbf{y}_{l}=\mathbf{W}^{d}_{l}(\text{SiLU}(\mathbf{W}^{g}_{l}\mathbf{x}_{l})\odot(\mathbf{W}^{u}_{l}\mathbf{x}_{l})),
$$

where $\odot$ denotes elementwise multiplication.

**Mixture of Experts (MoE).** In MoE, a dense FFN is replaced by $K$ expert FFNs $\{f_{l,k}\}^{K}_{k=1}$ and a router $r_{l}(\mathbf{x}_{l})$ that selects a small set $\mathcal{T}_{l}(\mathbf{x}_{l})$ of top-$r$ experts with mixture weights $\pi_{l,k}(\mathbf{x}_{l})$. Each expert use the same mechanism as normal FFN, but the layer output is

$$
\mathbf{y}_{l}=\sum_{k\in \mathcal{T}_{l}(\mathbf{x}_{l})}\pi_{l,k}(\mathbf{x}_{l})f_{l,k}(\mathbf{x}_{l}).
$$

**Hash-layer Mixture of Experts.** To eliminate _trainable routing and auxiliary loss_, hash-layer MoE fixes a balanced, token-id based mapping to experts. The FFN output becomes

$$
\mathbf{y}_{l}=\sum_{k\in \text{hash}(t)}f_{l,k}(\mathbf{x}_{l}).
$$

**Scaling the number of experts.** _Mixture of Word Embeddings_ pushes expert granularity to the word level, instantiating hundreds of thousands of small experts. This method increases model's _knowledge capacity_ and promotes _word-specific specialization_. However the extreme expert count magnifies **two well-known MoE-related challenges:**

- _Communication overhead._ We have more fragment peer to peer exchange, and communication becomes more _latency-dominated_ due to many small payloads and overhead but not bandwidth bottleneck.
- _Unbalanced expert frequency._ Word frequencies are [Zipfian](https://en.wikipedia.org/wiki/Zipf%27s_law), so only a few high-frequency experts receive disproportionate traffic while a long tail is rarely activated.

**Per Layer Embedding.** Unlike MoWE, the Per Layer Embedding (PLE) _share_ the gate projection and down projection of the FFN block across expert sub-networks. However they didn't completely dispense the existing FFN in each decoder layer. Instead they append an additional PLE block to FFN. And the token-level specific mapping tables are not shared across multiple devices, but _stored on node-local CPU memory_. So they can be pre-fetched as required, thus avoid high all-to-all communication traffic. Sharing the gate and down projection also alleviates negative effects of frequency imbalance.

## Method

### STEM

STEM builds upon the design of PLE, but there are some differences:

1. PLE dose not completely dispense with the existing FFN block, while STEM do.
2. PLE embedding tables are usually much more low-dimensional compared the intermediate dimension of the regular FFN layers, but STEM has the same dimension.

For layer $l$, let $\mathbf{U}_{l}\in \mathbb{R}^{V\times d_{\text{ff}}}$ be the per-layer embedding table. Given input $\mathbf{x}_{l}\in \mathbb{R}^{d}$, the STEM layer computes

$$
\mathbf{y}_{l}=\mathbf{W}^{d}_{l}(\text{SiLU}(\mathbf{W}^{g}_{l}\mathbf{x}_{l})\odot\mathbf{U}_{l}[t]),
$$

where $\mathbf{U}_{l}[t]\in \mathbb{R}^{d_{\text{ff}}}$ is the row of $\mathbf{U}_{l}$ corresponding to token $t$.

### Insights

This part makes an attempt to realize the full potential of the layer-specific embedding tables unlike PLE.

They view FFNs as a key-value query operation, where $\mathbf{W}^{u}$ is key, $\mathbf{W}^{d}$ is value, and $\mathbf{W}^{g}$ makes it _query-dependent_, yielding sharper, context-adaptive retrieval than a single stream FFN.

So why STEM choose to replace up projection with token id embedding but not other two projection? This is because static embedding lacks _context information_, while gate projection is responsible for this, so we should not replace it. And if we replace down projection, it means we are going to make the value static, and breaks the model's forward path (we know id then know the output, no need of computation!).

### Knowledge Editing

Since each token has their own embedding, if we change the embedding value of specific id, it can perform very differently. If we replace America with France embedding, then ask questions like what is the capital of America, the model will answer Paris. The details about how to deal with token count mismatch are in the paper.

### System Implementation

Naive implementation of STEM can introduce system challenges. The STEM embedding table size grows linearly with vocab size, FFN intermediate dimension, and number of STEM layers. The key optimizations include _parallel embeddings, CPU offloading, asynchronous computation and communication, token deduplication, and LFU caching_.

During inference, we offload the large STEM embedding tables to CPU. Because the STEM embeddings are indexed by input token ids, so they can be prefetched asynchronously. Since input token follows a Zipfian distribution, we can use a memory efficient LFU cache to increase hit rate.

The distributed training strategy is put a full STEM table on each node, shall be introduced more concretely in future works.s

# Third Pass

Note that, STEM is not conflict with MoE, because STEM is a FFN replacement method, and MoE is actually a lot of FFNs, they are orthogonal approaches to scale up model paramters. So each FFN in MoE can also be STEM, then we get a _Mixture of STEM Experts_.
