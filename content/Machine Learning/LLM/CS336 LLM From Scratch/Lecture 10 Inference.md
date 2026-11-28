# Review Architecture

[[Transformers and LLM]]

There are two stages of inference:

- **Prefill**: given a sequence, encode tokens in parallel, which is compute limited
- **Generation**: generate one token at a time, which is memory limited

Compute limited is good because we use up all of our compute resources, but memory limited is bad. So the main bottleneck is memory. Let's see how to mitigate it.

# Taking Shortcuts (lossy)

We want to further improve model inference throughput or latency based on limited given resources, since we have already pushed batch size or sequence length to the limit, we must pay extra price for the power.

## Reduce Key Value Cache Size

Recall that memory is the bottleneck for inference, so let's reduce the size of KV cache, but make sure we don't lose much accuracy.

### GQA

Group Query Attention, for a group of queries, now we only have 1 key value cache. Reduce memory cost significantly.

### MLA

Project each key and value vector from $N\times H$ (head numbers and head dimensions) dimensions to $C$ dimensions. A wrinkle is: MLA is not compatible with RoPE, so we have to add additional 64 dimensions for RoPE.

### Local Attention

The idea is just look at the local context, which is most relevant. This turns out to hurt accuracy, we can interleave local attention with global attention.

In summary, the goal is to reduce KV cache size without hurting accuracy. So we have lower-dimensional KV cache or use local attention on some of the layers.

## Get Rid of Transformer

### State-Space Models

[[Mamba]]

### Diffusion Models

[[Diffusion Model]]

## Quantization

[[Quantization - TinyML]]

## Pruning

[[Pruning and Sparsity - TinyML]]
After pruning we usually do distillation to learn from itself then iterate.

## Speculative Sampling

Remember that we have proposed, generation is slower than prefilling, because the later one can parallel, so we can modify the generation a little bit, which is using a cheaper **draft model** to guess a few tokens a head of time (because they are smaller and faster), then evaluate with **target model**, so that it can process tokens in parallel and accept it if looks good. This makes the model behaves like generating multiple tokens at a time.
