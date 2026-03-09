Paper link: [Paper](http://arxiv.org/abs/2512.02556)

**Prototype of DSA.** DSA has two components: a lightning indexer and a fine-grained token selection mechanism.

The **lightning indexer** computes the index score $I_{t,s}$ between the query token $\mathbf{h}_{t}\in \mathbb{R}^{d}$ and a preceding token $\mathbf{h}_{s}\in \mathbb{R}^{d}$, determining which tokens to be selected by the query token:

$$
I_{t,s}=\sum ^{H^{I}}_{j=1}w^{I}_{t,j}\cdot \text{ReLU}(\mathbf{q}^{I}_{t,j}\cdot \mathbf{k}^{I}_{s}),
$$

where $H^{I}$ denotes the number of indexer heads; $\mathbf{q}^{I}_{t,j}\in \mathbb{R}^{d^{I}}$ and $w^{I}_{t,j}\in \mathbb{R}$ are derived from the query token $\mathbf{h}_{t}$; and also does key (preceding) token. Use ReLU for throughput consideration, we still need to do attention.

Given the index scores $\{I_{t,s}\}$ for each query token $\mathbf{h}_{t}$, our **fine-grained token selection mechanism** retrieves only the key-value entries $\{\mathbf{c}_{s}\}$ corresponding to the top-k index scores. Then the attention output $\mathbf{u}_{t}$ is computed by applying the attention mechanism between the query token $\mathbf{h}_{t}$ and the sparsely selected key-value entries $\{\mathbf{c}_{s}\}$:

$$
\mathbf{u}_{t}=\text{Attn}(\mathbf{h}_{t}, \{\mathbf{c}_{s}\mid I_{t,s}\in \text{Top-k}(I_{t,:})\}).
$$

**Instantiate DSA under MLA**. MLA comes from DeepSeek's previous work, therefore they implement DSA based on the MQA mode of MLA, where each latent vector (the key-value entry of MLA) will be shared across all query heads of the query token.

# Continued Pre-Training

Starting from a _base checkpoint_ of DeepSeek-V3.1-Terminus, they continued pre-training followed by post-training to create DeepSeek-V3.2.

**Dense Warm-up Stage.** First we use a short warm-up stage to initialize the lightening indexer. In this stage, we keep dense attention and freeze all model parameters _except for the lightning indexer_. To align the indexer outputs with the main attention distribution, for the $t$-th query token, we first aggregate the main attention scores by summing across all attention heads. Then produce a _L1-normalized_ target distribution $p_{t,:}\in \mathbb{R}^{t}$, along the sequence dimension. Based on $p_{t,:}$, we set a _KL-divergence loss_ as the training objective of the indexer:

$$
\mathcal{L}^{I}=\sum_{t}\mathbb{D}_{\text{KL}}(p_{t,:}\|\text{Softmax}(I_{t,:})).
$$

**Sparse Training Stage.** After indexer warm-up, we do the fine-grained token selection mechanism and optimize all model parameters. In this stage, we also _keep aligning_ the indexer outputs to the main attention distribution, but considering _only the selected token set_ $\mathcal{S}_{t}=\{s\mid I_{t,s}\in \text{Top-k}(I_{t,:})\}$:

$$
\mathcal{L}^{I}=\sum_{t}\mathbb{D}_{\text{KL}}(p_{t,\mathcal{S}_{t}}\|\text{Softmax}(I_{t,\mathcal{S}_{t}})).
$$

They _detach the indexer input from the computational graph_ for separate optimization. The training signal for indexer is from only $\mathcal{L}^{I}$, while the optimization of the main model is according to only the language modeling loss.

# Parity Evaluation

**Standard Benchmark** Comparing to DeepSeek-V3.1-Terminus, V3.2 Exp significantly improves computational _efficiency on long sequences_, and do not observe substantial performance degradation on both short- and long-context tasks.

**Human Preference** Given that direct human preference assessments are inherently susceptible to bias, they employ ChatbotArena as an indirect evaluation framework to approximate user preference for the newly developed base models. The result shows that the new base model performs even better than old one.

**Long Context Eval** Result shows that V3.2 does not regress on long context tasks.

# Inference Cost

DSA reduces the core attention complexity of the main model from $\mathcal{O}(L^{2})$ to $\mathcal{O}(Lk)$, where $k$ ($\ll L$) is the number of selected tokens. Although lightning indexer still has a complexity of $\mathcal{O}(L^{2})$, it requires much less computation compared with MLA. Constants matter! Combining with their optimized implementation, DSA achieves a significant end-to-end speedup in long-context scenarios.
