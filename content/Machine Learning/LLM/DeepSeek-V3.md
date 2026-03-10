http://arxiv.org/abs/2412.19437

# Introduction

V3 is both powerful and economical. It proposed an FP8 mixed precision training framework and proves it to be useful on extremely large-scaled model.

# Architecture

The basic architecture of V3 is featured by **Multi-Head Latent Attention (MLA) for efficient inference** and **DeepSeekMoE for economical training**. Then they present a Multi-Token Prediction (MTP) training objective.

## Basic Architecture

The basic architecture is still within the Transformer framework. But they use Multi-Head Latent Attention instead of the standard Self-Attention, and use DeepSeekMoE as the Feed-Forward Network.

### Multi-Head Latent Attention

The core of MLA is the low-rank joint compression for attention keys and values to **reduce [[KV Cache|Key-Value (KV) cache]] during inference**. They first **down project the input to a lower dimension intermediate vector to cache**, and then up project it back to context length by head number size to calculate, which reduces the amount of data need to cache during generation.

### DeepSeekMoE with Auxiliary-Loss-Free Load Balancing

#### Basic architecture of DeepSeekMoE

Compared with traditional MoE like GShard, DSMoE uses fine-grained experts and isolates some experts as **shared** ones. For the shared ones, they are always used in the FFN, and for the normal ones, they use the experts with top-K token-to-expert affinity according to the token is under calculation. V3 uses the **sigmoid** function to compute the affinity scores, and applies a normalization among all selected affinity scores to produce the gating values.

#### Auxiliary-Loss-Free Load Balancing

An unbalanced expert load will lead to **routing collapse** and **diminish computational efficiency** in scenarios with expert parallelism (usually we want all experts to be visited). They balanced this by **introducing a bias term** $b_{i}$ for each expert and to the affinity scores $s_{i,t}$ to determine the top-K routing. Note that the **bias is only used for routing**, but the gating value still uses the original $s_{i,t}$. How does this work, if we are adding bias to all the experts? Don't worry! At the end of each step, we will decrease the bias term by $\gamma$ if its corresponding expert is overloaded, and increase it by $\gamma$ if its corresponding expert is underloaded.

#### Complementary Sequence-Wise Auxiliary Loss

To prevent extreme imbalance within any single sequence, they also employ a complementary sequence-wise balance loss:

$$
\begin{align}
\mathcal{L}_{\text{Bal}} &=\alpha \sum_{i=1}^{N_{r}}f_{i}P_{i}, \\
f_{i}&= \frac{N_{r}}{K_{r}T}\sum_{t=1}^{T}\mathbb{1}(s_{i,t}\in \text{Topk}(\{s_{j,t}|1\leq j\leq N_{r}\},K_{r})), \\
s_{i,t}'&= \frac{s_{i,t}}{\sum_{j=1}^{N_{r}}s_{j,t}}, \\
P_{i}&= \frac{1}{T}\sum_{t=1}^{T}s_{i,t}'
\end{align}
$$

#### Node-Limited Routing

There are several experts under single node, to avoid too much communication overhead, they introduced a **node limitation** $M$, each token will be sent to at most $M$ nodes, which are selected **according to the sum of highest $\frac{K_{r}}{M}$ affinity scores** of experts distributed on each node.

#### No Token-Dropping

No token is dropped during training and inference.

## Multi-Token Prediction

Use multiple tokens as input and output the same amount of tokens sequentially.

#### MTP Modules

Their MTP implementations uses $D$ sequential modules to predict $D$ additional tokens. The $k$-th MTP module consists of a shared embedding layer, a shared output head, a Transformer block and a projection matrix. For $i$-th token they first concatenate the previous depth representation with current embedding output to get a $2d$ vector, then project it back to $d$ dimension. After that they apply transformer and output head as usual.

#### MTP Training Objective

$$
\mathcal{L}_{\text{MTP}}^{k}=\text{CrossEntropy}(p^{k}_{2+k:T+1},t_{2+k:T+1})=- \frac{1}{T}\sum_{t=2+k}^{T+1}\log P_{i}^{k}[t_{i}]
$$

where $T$ denotes the input sequence length, $t_{i}$ denotes the ground-truth token at the $i$-th position, and $P_{i}^{k}[t_{i}]$ denotes the corresponding prediction probability of $t_{i}$, given by the $k$-th MTP module. Finally compute the average of the MTP loss across all depth and multiply it by a weighting factor $\lambda$ to obtain overall MTP loss:

$$
\mathcal{L}_{\text{MTP}}= \frac{\lambda}{D}\sum_{k=1}^{D}\mathcal{L}_{\text{MTP}}^{k}
$$

During inference they discard the MTP modules, the main model can function independently and normally.

# Infrastructures

## Compute Clusters

## FP8 Training

They propose a fine-grained mixed precision framework utilizing the FP8 data format for training V3. The main problem of low precision training is that it is hard to manipulate outliers for limited dynamic range. So to extend the dynamic range of the FP8 format, they introduce a **fine-grained quantization strategy**: tile-wise grouping or block-wise grouping, rather than use a single scale factor over the whole tensor.

### Mixed Precision Framework

Most compute-density operations are concluded in FP8, while a few key operations are strategically maintained in their original data formats to balance training efficiency and numeric stability. Can see the detail in the paper's figure.

### Improve Precision from Quantization and Multiplication

#### Fine-Grained Quantization

As a standard practice, the input distribution is aligned to the representable range of the FP8 format by scaling the maximum absolute value of the input tensor to the maximum representable value of FP8. This makes low-precision training highly sensitive to activation outliers. To solve this they introduced fine-grained quantization which **group some elements together and update the scales according to groups**.

#### Increasing Accumulation Precision

Instead of using limited bit width all the time, they use **limited bit width for intermediate results** and when an **interval of $N_{C}$ is reached**, these partial results will be copied to FP32 registers and **perform full-precision FP32 accumulation**.

## Inference and Deployment

### Prefilling

To achieve load balancing among different experts in the MoE part, we need to ensure that each GPU processes approximately the same number of tokens. They do so by adding _redundant experts_, which duplicates high-load experts and deploys them redundantly.

### Decoding

During decoding, they consider the shared experts as a "big" routed expert and it is a heavy-load one that will always be selected.
Similar to prefilling, they periodically determine the set of redundant experts in a certain interval.

## Suggestions on Hardware Design

All the suggestions are based on their implementation, and I have heard some Chinese companies are going to develop their next generation of chips base on such design.
