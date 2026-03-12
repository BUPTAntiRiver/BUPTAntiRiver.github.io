Low-Rank Adaption, which freezes the pre-trained model weights and injects trainable rank decomposition matrices into each layer of the Transformer architecture, in order to greatly reduce the number of trainable parameters for downstream tasks.

# Introduction

Do we need to fine-tune all the parameters? How expressive should the matrix updates be, in other words, what should be its **rank**?

LoRA appears at the age when _adaption to varying kinds of down-stream tasks is very popular_. And the adaption is usually done with _fine-tuning_, which updates all parameters of the pre-trained model. LoRA can be seen as a generalization of full fine-tuning, which fine-tune part of the parameters and fine-tune part of the ranks.

Previous methods often introduce inference latency.

Their method has several key advantages:

1. A pre-trained model can be shared and used to build many different LoRA models.
2. LoRA makes training more efficient.
3. No inference latency.
4. Orthogonal to many prior methods and can be combined with many of them.

# Method

## Low-Rank-Parameterized Update Matrices

Instead of updating the $d\times d$ weights, we update a low-rank version $W_{0}+\Delta W=W_{0}+BA$ where $W_{0}$ is the pre-trained weights and $B\in \mathbb{R}^{d\times r},A\in\mathbb{R}^{r\times d}$ and the rank $r\ll \min(d,k)$. This showcases their idea of **generalization of full fine-tuning**. When $r=d$ and apply this to all parameters, we get the full version.

**No Additional Inference Latency.** When we want the model switch to a different downstream task, we can subtract $BA$ and add up $B'A'$ to get a new fine-tuned version of model, this only takes very little memory overhead.
