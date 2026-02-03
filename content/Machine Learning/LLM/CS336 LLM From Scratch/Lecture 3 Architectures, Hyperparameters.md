## Architectures

### [[RMSNorm]]

Let's look at the popular RMSNorm, it simply removed the bias term and mean term comparing to layer norm but it keeps the same performance. It reduced some FLOPs but the key part is it reduced memory usage of bias and mean.
This is because though FLOPs of normalization takes only a tiny part of total FLOPs, with less than 1%, but the run time we need for normalization can take 25%. A lot of memory transfer lead to such result.

### More generally

Most modern transformers don't have bias terms in order to use less memory and improve optimization stability.

### Activations

We have a whole zoo of activations now: ReLU, GeLU, SwiGLU, etc. What are they? What is the difference? Does it matter?

Most modern models used _gated activations_ (GLU).

GLUs modify the first part of a feed forward layer:

$$
\text{FF}(x)=\max(0, xW_{1})W_{2}
$$

Instead of a linear + ReLU, augment the above with an (entriywise) linear term:

$$
\max(0, xW_{1})\to\max(0,xW_{1})\otimes(xV)
$$

This gives the gated variant, notice that we have an extra parameter $V$.

So the difference between GeGLU and SwiGLU is that they use different function for non-linearity.

I am going to present the activation used by SwiGLU here:

$$
\begin{align}
\text{swish}(x)&=x\sigma(\beta x) \\
\sigma (x)&= \frac{1}{1+e^{-x}}
\end{align}
$$

### Position Embedding

Sine embedding, absolute embedding, relative embedding, etc.

Nowadays, seems RoPE has won the game. Its idea is that we only care about the relative distance between two tokens, and it is very similar to inner product of two vectors, if the angle between two vectors stays the same, then their inner product won't change. They turn the offset of position into rotation, so that if the relative distance of two token does not change, their embedding relation won't change. It makes the attention weight depending (only) on position distance.

The rotation in 2 dimension is straight forward, but our token has huge embedding dimension, so the RoPE inventors find an simple way which is breaking down the token vector into pieces of 2 dimension and embed them separately.

### Attention Heads

#### GQA/MQA

Let's think about the compute involved for attention. Total arithmetic operations is $bnd^{2}$, total memory access is $bnd+bhn^{2}+d^{2}$, for input transform, softmax and projection. We can infer that _arithmetic intensity_ is high $O\left( \left( \frac{1}{k} + \frac{1}{bh} \right)^{-1} \right)$, so we can keep our GPUs running.
But this is different at inference time. We cannot parallel the generation process, in this case we need to recompute/update attention via _KV cache_. Total arithmetic operations is still $bnd^{2}$, but memory access becomes $bn^{2}d+nd^{2}$ and arithmetic intensity becomes $O\left( \left( \frac{n}{d} + \frac{1}{b} \right)^{-1} \right)$, so we need large batches or short sequence length and large model. However the model structure is hard to change.
The key idea of MQA (multi-query attention) is to have multiple queries but just one dimension for keys and values in order to reduce KV cache. GQA (group-query attention) is not as radical as MQA that reduce head dimension of KV to only 1, but has some key-query ratio like 2 query 1 key.

#### Sparse Attention & Sliding Window Attention

Reduce expressiveness in order to improve runtime.

#### More Modern

Interleaving of full attention and low-rank attention.

## Hyperparameters

### Some Ratioes

The dimension of FF layer is usually 4 times of attention output for non-gate activation and $\frac{8}{3}$ which is around 2.6 for gate activation, because we have additional parameter for gating.
And in multi-head attention, the typical method is separating the model dimension to each head rather than letting each head has the same dimension as model dimension. The common choice of ratio between `num_head * head_dim` and `model_dim` is 1.
**Aspect ratio**, ratio of model dimension and number of layers, the common choice is around 100.

### Regularization

Do we really need regularization in pretraining? Because in modern large language model training, we have trillions of pretrain data, and we usually train it with a single epoch, so it is not going to overfit easily. Actually, in modern models, people use low dropout rate or just don't use it during pretraining. However weight decay is still in use, it has more complex effects.
