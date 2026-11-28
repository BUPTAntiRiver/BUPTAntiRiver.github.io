Resources: [huggingface blog](https://huggingface.co/blog/gemma4)

Gemma 4 supports image, text and audio inputs, and generates text responses. And it is the newest released open source model, which is said to be more powerful than model with 10 times or 20 times size. But that score is on model arena, rather than some solid benchmark, so I don't buy that.

Since it is open source, let's dig in what new methods it has brought to the world.

Gemma 4 has four different models, two are embedding models, one dense model with most powerful performance and one MoE.

# Architecture

The main architectures characteristics in Gemma 4 are:

- Alternating **local sliding-window** and **global full-context** attention layers. Smaller dense models use sliding windows of 512 tokens while larger models use 1024 tokens.
- **Dual RoPE** configurations: standard [[RoPE]] for sliding layers, proportional RoPE for global layers, to enable longer context.
- **Per-Layer Embeddings ([[Scaling Transformer with Embedding Modules|PLE]])**: a second embedding table that feeds a small residual signal into every decoder layer. (I put the link of STEM here because STEM is built upon the idea of PLE)
- **Shared KV Cache**: the last N layers of the model reuse key-value states from earlier layers, eliminating redundant KV projections.
- **Vision encoder**: uses learned 2D positions and multidimensional RoPE. Preserves the original aspect ratios and can encode images to a few different token budgets (70, 140, 280, 560, 1120).
- **Audio encoder**: USM-style conformer with the same base architecture as the one in Gemma-3n.

## Per-Layer Embeddings (PLE)

In a standard transformer, each token gets a single embedding vector at input, and the same initial representation is what the residual stream builds on across all layers, forcing the embedding to forward everything the model might need.

PLE adds a parallel, lower-dimension conditioning pathway alongside the main residual stream. For each token it produces a small dedicated vector for every layer by combining _two signals_: a token _identity_ component (from an embedding look up), and a _context-aware_ component (from a learned projection of the main embeddings).

Then each decoder layer uses its corresponding PLE vector to modulate the hidden states with a residual block after attention and FFN.

You can figure out that PLE is also a method that increases model size while only importing a little computation overhead.

## Shared KV Cache

This is also a method that improves model efficiency but has minimal harm to quality. It means that the last `num_kv_shared_layers` layers of the model don't compute their own key and value projections. _Instead_, they _reuse_ the $K$ and $V$ tensors from the last non-shared layer with the same attention type (sliding or full).

# Performance

Let's look at the real benchmark but not the boasted model arena ELO score, and compare with the Qwen3.5-397B many medias said was defeated by Gemma 4.

Qwen source: https://qwen.ai/blog?id=qwen3.5. Gemma 4 source is the same as top.

| Benchmark  | Gemma 4 31B | Qwen3.5 397B |
| ---------- | ----------- | ------------ |
| MMLU-Pro   | 85.2        | 76.01        |
| MMMLU      | 88.4        | 85.82        |
| MMMU Pro   | 76.9        | 79.0         |
| MathVision | 85.6%       | 88.6         |

I have to say, the result blows my mind, Gemma 4 is indeed better and actually even it is worse we cannot blame it, because it is smaller, but it is even stronger. Oh my god, Google good job.

# 总结

其实我觉得，Gemma 4 也没什么创新点，技术用的都是现成的，为什么就会更强呢？说白了我觉得现在的模型最关键的还是数据。

打个比方，现在的模型就是学生，数据就是教材，基建是培训场所。从开源模型的观察来看，我们都没有用 full attention（Qwen3.5 GDN, Gemma 4 SWA），这就好比模型是资质一般的学生，但是他们教起来容易，教起来快，同时，各个大厂他们们内部的教育资源才是关键。模型不存在天才，必须要靠教，相同的模型，你给一个普通开发者用开源的各种数据自己清洗训练，和这些大厂内部持有的数据以及堆财力物力达成的高质量标注相比，肯定效果是不如的。

至于 Infra 这一块，主要还是迭代速度，和商业化更相关，与模型质量的关系稍低一些。

所以为什么 Gemma 4 这么强？依旧还是这些开源模型没有开源的数据最关键。包括当时读 DeepSeek-R1 论文的时候，里面一笔带过他们生产了很多高质量推理数据，只有一两句话，但我想这和他们提出的训练范式同样非常关键。
