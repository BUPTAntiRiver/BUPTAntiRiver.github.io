A wrong interpretation to scaling law is: scale is all that matters, algorithms don't matter.
Right interpretation: algorithms that scale is what matters.
Accuracy = Efficiency $\times$ Resources
So the question is, what is the best model one can build given a certain compute and data budget? Which is, _maximum efficiency_.

## Basics

### Tokenization

In this lecture, we introduce Byte-Pair Encoding (BPE) tokenizer.

### Architecture

The starting point is Transformer, but since 2017, we have done a lot of things to make it work better:

- Activation functions: ReLU to SwiGLU
- Positional encoding: sinusoidal to [[RoPE]]
- Normalization: LayerNorm to [[RMSNorm]]
- Placement of normalization: pre-norm versus post-norm
- MLP: dense to [mixture of experts](MoE-Mixture-of-Experts.md)
- Attention: full, sliding window, liner, flash
- Lower-dimensional attention: group-query attention ([[Transformers and LLM#^GQA|GQA]]), multi-head latent attention ([[DeepSeek-V3#Multi-Head Latent Attention|MLA]])
- Challengers: state space models like [[Mamba]]

## Systems

We are going to talk about GPU.
And things like _Kernel_, _Parallelism_, _Inference_ (which is said to be the main consumption of computing power).

## Scaling Laws

Do experiments at small scale, predict hyperparameters/loss at large scale.

## Data

What capabilities do I want the model to do?
Wikipedia, GitHub, ArXiv, etc.

### Evaluation

Perplexity, instruction following, LM-as-a-judge, etc.

### Data Curation

Data does not fall from the sky, it has to be actively acquired somehow. It may be html, pdf or directories. Also some online public data requires license.

### Data processing

Transformation, filtering, deduplication, etc.

## Alignment

After all the previous training work, we are not getting a chat model, but a next-token predict model. We call it a _base model_, we need to get it good at:

- Following instructions
- Good response style
- Safety

Generally two phases:

- supervised fine tuning
- learn from feedback
