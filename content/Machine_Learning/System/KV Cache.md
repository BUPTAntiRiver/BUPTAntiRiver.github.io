# What?

KV Cache is a system optimization applied to [[Transformers and LLM|Transformer]], which is exactly the same to self-attention in math.

# Why?

KV Cache saves computation cost by cache the key and value matrices used in Transformer, which improves compute performance.

# How?

Suppose we have a sequence of tokens $s$ of length $N$. And we are going to do decoding with a decoder only model. Then in the compute trajectory of transformer, we have a **embedding layer** that maps token id sequence into embeddings. Now we have input tensor of shape $[N, d_{\text{embed}}]$

Skip the normalization for now, we goes into attention. We calculate our $Q,K,V$ which have shape of $[N, d]$ in total (after head concatenation), then pass into feed forward networks and repeat. Finally we still get an output tensor of shape $[N,d]$ with a final linear transform, we will get shape of $[N,V]$ where $V$ is the size of vocabulary to do softmax operation and guess the next token.

Wait, we have $N$ of such $V$, there will be $N$ guesses, what?

You are right, in training all these stuff matters, because given a known sequence of length $N$, we can generate $N$ next token prediction sample (each one guess its next). But for inference, we actually _only need the last token_ for query, but others are needed for $K,V$ calculation.

So this is where the idea of KV Cache comes from. When we have the new token and append it back to perform the next token prediction, we actually only need to compute the new token's query, key and value, then do some computation, because old queries are **unused** for inference and old keys and values are already calculated, which means we can cache them.

Though we might have pre-normalization or post-normalization difference, it does not performs on $N$ dimension, but on the last dimension. So new token won't affect old tokens' $K,V$ results.
