---
aliases:
  - RAG
---

# What is RAG?

RAG was created in order to solve knowledge intensive problems, like those problems that we need to search the Wikipedia. Though in traditional transformer models we can just fuse all knowledge into parameters without any outer information resource, however, when this augmented structure was added, model performs much better. It showed that we can expand and revise the knowledge easily, and accessed knowledge can be inspected and interpreted. There are also less "hallucinations".

Comparing to normal sequence to sequence model (seq2seq) with only **parametric** memory, RAG models has a seq2seq model as **parametric** model and a dense vector index of Wikipedia (any other knowledge resource can be OK, just need to rebuild the index) which can be accessed with a pre-trained neural retriever as **non-parametric** memory.

Their results highlight the benefits of combining parametric and non-parametric memory with generation for knowledge-intensive tasks. Also the non-parametric memory can be replaced to update the model's knowledge as the world changes.

# How to RAG?

RAG models uses input sequence $x$ to retrieve text document $z$ and use them as additional context when generating target sequence $y$.

![[RAG-Architecture.svg]]

RAG model mainly leverages two components:
1. a retriever $p_{\eta}(z|x)$ with parameters $\eta$ that returns (top-k truncated) distributions over text passages given a query $x$
2. a generator $p_{\theta}(y_{i}|x, z, y_{1:i-1})$ parameterized by $\theta$ that generates a current token based on the context of previous $i-1$ tokens $y_{1:i-1}$, the original input $x$ and retrieved passage $z$.

We consider the retrieved document as a latent variable. So we propose two models that marginalize over the latent variable in different ways to produce a distribution over generated text.
1. RAG-sequence, the model uses the same document to predict each target token.
2. RAG-token, can predict each target token based on a different document.

## Models

### RAG-Sequence Model

Uses the same document to generate complete *sentence*. The top $k$ documents are retrieved using the retriever, and the generator produces the output sequence probability for each document, which are then marginalized.

$$
	p_{\text{RAG-Sequence}}(y|x)\approx \sum_{z\in \text{top-}k(p(\cdot|x))}p_{\eta}(z|x)p_{\theta}(y|x,z)=\sum_{z\in \text{top-}k(p(\cdot|x))}p_{\eta}(z|x)\prod_{i}^{N}p_{\theta}(y_{i}|x,z,y_{1:i-1})
$$

### RAG-Token Model

Can draw a different latent document for each target *token* and marginalize accordingly. This allows generator to choose content from several documents while producing an answer. Unlike RAG-Sequence, we marginalize at every single token here.

$$
	p_{\text{RAG-Token}}(y|x)\approx \prod_{i}^{N}\sum_{z\in \text{top-}k(p(\cdot|x))}p_{\eta}(z|x)p_{\theta}(y_{i}|x,z,y_{1:i-1})
$$

## Retriever

The retriever component $p_{\eta}(z|x)$ is based on DPR in the paper. DPR follows a bi-encoder architecture:

$$
	p_{\eta}(z|x) \propto \exp(\mathbf{d}(z)^{\top}\mathbf{q}(x))
$$

where $\mathbf{d}(z)$ and $\mathbf{q}(x)$ are dense representations of document and input query produced by *document encoder* and *query encoder*, which are both based on BERT (but they are two distinct models).

## Generator

The generator component $p_{\theta}(y_{i}|x,z,y_{1:i-1})$ can be modeled with any encoder-decoder (now decoder only is more popular). They used BART-large in the paper.

## Training

Given a fine-tuning corpus of input/output pairs $(x_{j},y_{j})$, we minimize the negative marginal log-likelihood of each target, $\sum_{j}-\log p(y_{j}|x_{j})$ using stochastic gradient descent with Adam (we can use better methods now). Updating the document encoder during training is costly as it requires the document index to be periodically updated. They do not find this step necessary for strong performance, so they keep document encoder fixed and only update query encoder and generator.

## Decoding

At test time, RAG-Sequence and RAG-Token require different ways to approximate $\arg \max_{y}p(y|x)$.

### RAG-Token

Can be seen as a standard autoregressive seq2seq generator with transition probability: $p'_{\theta}(y_{i}|x,y_{1:i-1})=\sum_{z_{i}\in \text{top-}k(p(\cdot|x))}p_{\eta}(z_{i}|x)p_{\theta}(y_{i}|x,z_{i},y_{1:i-1})$. To decode, we can plug $p'_{\theta}(y_{i}|x,y_{1:i-1})$ into a standard beam decoder.

### RAG-Sequence

The likelihood $p(y|x)$ does not break into a conventional per-token likelihood, hence we cannot solve it with a single beam search. We need to care about the $(z,y)$ pairs, so we run beam search for each document $z$, scoring each hypothesis $y$ using $p_{\theta}(y_{i}|x,z,y_{1:i-1})$. This yields a set of hypothesis, we call it $Y$, and some of which may not have appeared in the beams of all documents. Let me explain this, suppose $z_{a}$ gets $y_{a}$ and $z_{b}$ gets $y_{b}$, we have result of $p(y_{a}|x,z_{a})$ and $p(y_{b}|x,z_{b})$ during beam search but to get marginalized result $p(y_{a}|x)$ and $p(y_{b}|x)$ we also need $p(y_{a}|x,z_{b})$ and $p(y_{b}|x,z_{a})$, so we need an additional teacher-forced run for all of these scenario. And now we can sum the probabilities across beams. We refer to this decoding procedures as "**Thorough Decoding**."

For longer output sequences, $|Y|$ can be very large, requiring many forward passes, so we have "**Fast Decoding**", which makes a further approximation that $p_{\theta}(y|x,z_{i})\approx 0$ where $y$ was not generated during beam search from $x,z_{i}$.

# Further Discussion

RAG is a prototype model, so how is it nowadays? With thriving agent works everywhere? How is RAG integrated into agent? It seems like we only see tool calls now.

# Materials

- https://arxiv.org/abs/2005.11401 — Lewis et al., NeurIPS 2020: the original RAG paper, pairing a seq2seq generator with a dense Wikipedia index.
- https://arxiv.org/abs/2004.04906 — Karpukhin et al., EMNLP 2020: DPR, the dense dual-encoder retriever that most RAG pipelines build on.
- https://arxiv.org/abs/2312.10997 — Gao et al., 2023: survey covering Naive, Advanced and Modular RAG, plus evaluation and open problems.
- https://arxiv.org/abs/2310.11511 — Asia et al., ICLR 2024: Self-RAG, which learns when to retrieve and critiques retrieved passages via reflection tokens.
- https://arxiv.org/abs/2307.03172 — Liu et al., TACL 2024: shows models use context unevenly, so passage position matters as much as retrieval quality.
