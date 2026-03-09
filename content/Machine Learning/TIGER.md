Paper link: http://arxiv.org/abs/2305.05065

# Introduction

TIGER stands for _Transformer Index for GEnerative Recommenders_.

# Related Work

**Semantic Recommenders.** The difference between TIGER and previous semantic recommenders is that models in previous works learn a high-dimensional embedding for each item and perform an ANN (Approximate Nearest Neighbors) in MIPS (Maximum Inner Product Search) space to predict the next item. In contrast, TIGER use Generative Retrieval to directly predict the semantic ID of the next item.

In a word, old work predicts new item by how close they are to current item, while TIGER does not make the assumption that the next item should be similar to current item, it predicts next item with generative method directly.

This does make sense, because sometimes we dislike the content and scroll the screen very fast.

**Semantic IDs.** The traditional way to identify items is more random, like language model tokenization, which gives items randomly-assigned discrete IDs. In TIGER, we use Semantic ID. They are learned based on the content information of the items.

**Generative Retrieval.** In the past, we get next item, which are related items by learning search indices. Generative retrieval is a recently developed approach for document retrieval, where the task is to return a set of relevant documents from a database.

# Proposed Framework

The proposed framework consists of two stages:

1. _Semantic ID generation using content features._ This involves encoding the item content features to embedding vectors and quantizing the embedding into a tuple of semantic codewords. The resulting tuple of codewords is referred to as the item's Semantic ID.
2. _Training a generative recommender system on Semantic IDs._ A Transformer model is trained on the sequential recommendation task using sequences of Semantic IDs.

## Semantic ID Generation

We first need to create a semantic embedding for each item. This can be done by using _encoder models_ to encode the item semantic contents (e.g. titles, descriptions or images) into embeddings. Then the semantic embeddings are used to generate a Semantic ID for each item.

_Semantic ID_ is defined to be a tuple of *codeword*s of length $m$. Each codeword in the tuple comes from a different _codebook_. So the number of items that the Semantic IDs can represent uniquely is the product of codebook sizes. There are many different methods to generate Semantic ID. We hope that _similar items should have overlapping Semantic IDs_.

In the paper, they use **RQ-VAE for Semantic IDs.** The VAE part means that we first quantize the input vector, then rebuild from that, and the difference between rebuilt one and original one will be the loss. RQ stands for residual quantization, we quantize the input (index 0), then subtract the 0-th quantization result from the input, and feed that for next step of quantization. Each quantize result takes a place in the codebook, this is where the sequence of codewords comes from. This recursive approach approximates the input from a coarse-to-fine granularity.

**For collisions**, we append a new column of codeword at the end of Semantic ID, make sure each item has unique Semantic ID.

## Generative Retrieval with Semantic IDs

We construct item sequences for every user by sorting chronologically the items they have interacted with. Then given sequence (item-1 to item-n), we replace items with $m$-length Semantic ID, it becomes $(c_{1,0},\dots, c_{1,m-1},\dots,c_{n,0},\dots, c_{n,m-1})$. The model is then trained to predict the Semantic ID of item-n+1. There might be cases that _invalid_ Semantic IDs are generated, but statistics show that it rarely happens, also they can handle it with methods like discard or beam search filtering.

# Simple Reimplement

I vibecoded a simple reimplement project, it performs a bit better that direct embedding similarity search, but the parameters I used wasn't well tuned, and the size of model is too small, and the coding agent has some illusion, which says paper Semantic ID size is only 10,000. So the simple reimplement do show that TIGER should be the SOTA of that time.
