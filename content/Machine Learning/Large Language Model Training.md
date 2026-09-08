LLM nowadays are really really large, even hard to fit in a 100 GB GPU! Because it usually needs TBs of memory to train long context model.
The cost of memory makes up of these parts:

1. parameters
2. gradients
3. optimizer intermediate values
4. activations
   Activations are usually the largest component.

# DeepSpeed

This is a method produced by Microsoft, which can reduce parameters, grads, optimizer size on multiple GPU by slicing them, and it is really helpful! But this is just a start. Because these guys grow **linearly** with context length, but activation grows **quadratic** on current transformer based models. For the matrix multiplication in attention. So the huge overhead of activation is not solved, and that's why [[FlashAttention]] comes.

# FlashAttention

Check [[FlashAttention]].

# How are large language models trained with multiple devices?

Large language models are too big for a single device to train, so we need **distributed training** to divide **the model, data and computation** across multiple devices, then synchronize updates. This is kind of something we have introduced in DeepSpeed.
[[Distributed Training - TinyML]]
