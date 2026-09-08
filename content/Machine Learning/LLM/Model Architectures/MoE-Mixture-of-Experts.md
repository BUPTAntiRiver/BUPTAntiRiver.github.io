reference: [Mixture of Experts Explained](https://huggingface.co/blog/moe)

# What?

MoE is in the context of **Transformer** models, and a MoE has 2 main elements:

- **Sparse MoE layers**: which replace FFN (Feed Forward Network) layers, MoE layers has a certain number of "**expert**s", each expert is a neural network. In practice they can be FFNs or more complex networks like MoE itself, which leads to hierarchical MoEs.
- **Gate Network or Router**: which determines which tokens are sent to which expert(s).

## What is **sparsity**?

Sparsity uses the idea of **conditional computing**. In dense model all parameters are used for all the inputs, while sparsity allows MoE to only run some part of the whole system. So we can increase model size with same compute.

### Challenges

But this introduces some challenges, for example, batched data are widely used, given we have a batch of inputs with 10k tokens, but 9k of them were sent to one expert, the others are sent to different experts, leading to **uneven batch sizes and underutilization**.

### Solution

A learned gating network (G) decides which experts (E) to send a part of the input:

$$
y=\sum_{i=1}^nG(x)_{i}E_{i}(x)
$$

Now all experts run for all inputs - it is a weighted multiplication.
What's a typical gating function? In the most traditional setup, we just use a simple network with a softmax function. The network will learn which expert to send the input.

$$
G_{\sigma}(x)=\text{Softmax}(x\cdot W_{g})
$$

There are some other techniques, like Shazeer's Noisy Top-k Gating.

#### Load balancing tokens for MoEs

In normal MoE training, the gate network converges to mostly the same few experts. This self-reinforces as favored experts are trained faster and hence selected more. To mitigate this, an **auxiliary loss** is added to encourage giving all experts equal importance. Check the [[DeepSeekMoE|DeepSeek]] papers to see their design.

# Why?

MoE has **better compute efficiency**, which means you can **scale up** model size or dataset size with the _same compute budget_ as a dense model.
Although MoEs provides efficient pretraining and faster inference compared to dense models, they also come with challenges:

- **Training**: struggled to generalize during fine-tuning, leading to overfitting.
- **Inference**: although a MoE model might have a lot of parameters, but its inference is pretty fast, because during inference only some of the experts are used. However all parameters need to be loaded in RAM, but actually it would need less space than number of experts times size of an expert, because there are some shared layers.

## MoEs vs dense models

Experts are useful for high throughput scenarios with many machines. Given a fixed compute budget for pretraining, a sparse model will be more optimal. For low throughput scenarios with little VRAM, a dense model will be better.

# How?

## MoEs and Transformers

Use [GShard](https://arxiv.org/abs/2006.16668) developed by Google as an example. GShard replaces every FFN with MoE using top-2 gating in both the encoder and the decoder. To **maintain a balanced load and efficiency** at scale, GShard introduced some changes:

- **Random Routing**: in a top-2 gating, we always pick the top expert but the second expert is picked randomly with probability proportional to its weight.
- **Expert Capacity**: set a threshold for how many tokens an expert can process.

## Router Z-loss

MoE has a lot of benefits, but it also struggles with training and fine-tuning instabilities.
We can use many methods to stabilize sparse models at the expense of quality. For example, introducing dropout improves stability but leads to loss of model quality.
Router Z-loss is, introduced in [ST-MoE](https://arxiv.org/abs/2202.08906), improves stability without quality degradation by **penalizing large logits entering the gate network**.

## What does an expert learn?

The encoder experts specialize in a group of tokens or shallow concepts. On the other hand, the decoder experts have less specialization.

## Fine-tuning MoE

Sparse models are more prone to overfitting, so we can explore higher regularization (e.g. dropout) within the experts themselves (we can have higher dropout rate for sparse layers and normal on for dense layers).
Another method is only freezing the MoE layers, by doing so, we can speed up training while preserving the quality.
Instruction Tuning also appears to benefit MoEs more than dense models.

## Making MoEs go brrr

MoE used to be hard to train, but now many methods have been developed to speed up. That makes MoE goes brrrrrrrrrrrrrrrrrr.

### Parallelism

Check this note for some common distributed training methods nowadays: [[Distributed Training - TinyML]].
We can assign each expert a device for parallel training.

### Serving Techniques

A big downside of MoE is the large number of parameters. Don't worry, we can use [[Knowledge Distillation(KD) - TinyML]] to learn a smaller version.
