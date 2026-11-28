Transformers is the most widely used architecture in generation AI field nowadays, but it is not as complex and difficult as its name. In this article we will dive into its details and introduce all newly established methods to make it even better.

# Intro

Transformers was initially introduced in sequence generation tasks. First I want to explain why transformers makes such a huge difference. It is because it beats all methods before it greatly, so what are they and why?

The two main methods in pre-transformers era are **Convolution Neural Networks (CNN)** and **Recurrent Neural Networks (RNN)**. We are not going to introduce their details here, but we want to mention their **disadvantages**. So CNN has limited context window, which means it only perceives limited information from previous layer at each point. And RNN has the problem of invisibility to early information in sequence, because the original information was processed every time we move on to the next word, so how many words we have, how many times the original information is modified or we can say disturbed, so later generated words has few connection to early words but that could be possible in reality.

While, Transformers do solve the problem that it can access whole sequence (CNN problem) also access them very easily (RNN problem).

# Algorithm

In this part we will introduce the full trace of a sequence becomes tokens, and becomes embedding, then go through the stacking transformer blocks and finally predicts the next token.

## Tokenizer

First of all, what is **token**? To solve this question, we need to understand how is a sequence represented in computer. Obviously they are composed of bits, zeros and ones, since they are ASCII codes, so we can represent a sequence as a vector of zero or one. So in this case a token is a zero or one. Token is the minimal representation level in our sequence.

But this idea is very costly, since we expands a sequence to a much longer sequence and that adds much more computation burden, also there is not very much information carried in such form.

So the next common idea is we are giving every character an ID index. That would be OK if we only consider English, but there are many other languages that has a lot of characters. This is a decent method but can we do even better?

The modern popular method is called Byte Pair Encoding. Which means we go back to byte level token but it is more flexible, we wrap pairs of bytes into one token. So the compress ratio (average bytes in one token) can be higher. Then how are these pairs decided? It is based on the tokenizer training dataset. We first have a basic token vocabulary that stands for 256 possible bytes and some special tokens like "<|endoftext|>" which has semantic information. Then we conduct the frequency of all token pairs in dataset, and merge the most frequent pair to a new token and append it to our vocabulary. We repeat this until we reach desired vocabulary size. The merged pairs can be merged again, so that we may have a token represents a lot of bytes (maybe a long word).

## Model Architecture

### Embedding Layer

So now we have a BPE tokenizer, and it can turn all common text into token ID sequences now, which is the training input to our model. But currently token IDs are discrete numbers, which are very hard to process, so we have a embedding layer that maps IDs into continuous space.

There are many old-school word embedding methods, but in LLM the embedding matrix $E\in \mathbb{R}^{V\times d}$ is just also a trainable parameter, tuned in the training process.

### Transformer Block

In deep learning we stack blocks of layers, in LLM we stack transformer block. It is consisted of an Attention Layer and a Fully Connected MLP layer.

#### Attention Layer

This is the **core** new idea presented in transformers. Suppose we have embedded input $x\in \mathbb{R}^{N\times d_{\text{embed}}}$ where $N$ is token sequence length and $d$ is the hidden dimension. We will have three projection matrix, $Q,K,V\in \mathbb{R}^{d_{\text{embed}}\times d}$ that projects $x$ so we have $q,k,v\in \mathbb{R}^{N\times d}$ now. Then we compute the attention score:

$$
	s=\text{Softmax}\left( \frac{qk^{\top}}{\sqrt{ d }} +m\right)v
$$

The similarity between query and key selects the value. Very straight forward in intuition. Since normal LLM does a auto regressive generation, so current score is $s\in \mathbb{R}^{N\times d}$, which is the result for next token in every position of the sequence. To disable getting information from position after current position, we will apply attention mask, it is added to the result before softmax $m\in \mathbb{R}^{N\times N}$, the top right corner are all negative infinity and the rest are all zero. So the final result won't select values from future tokens (it is just telling you the right answer!).

##### Decoder And Encoder

The attention presented above is actually called decoder only, which means we are only doing next token generation so we mask future values. But actually we can have transformers to do other work like negative positive classification stuff, and we need no causal relationship, so we can remove the mask and enable every token sees other tokens.

When a model has both encoder and decoder, we might have cross attention, which uses result from encoder as Keys and Values and result from decoder as Queries.

Imagine a translation task, given context "I like cats" to encoder, and decoder starts to query it and generates translated result "我喜欢猫". When generating "喜欢" decoder has access to "I like cats" and only "我", quite makes sense.

#### Fully Connected Layer

Just simple FC layers to add more complexity and expressiveness to the model, also with some activation functions.

#### Normalization

We may apply normalization methods like layer norm or [[RMSNorm]].

# Development

Current works are done on aspects like how to speed up attention ([[FlashAttention]])? Is there replacement for full attention ([[Linear Attention]], [[Mamba]], [[GDN - Gated Delta Networks]])? Other training methods beyond simple cross entropy loss between ground truth and generated result (SFT)? We have other training methods (RL [[Post-training]]), but how to make it even better? Models becomes bigger how to train them ([[Distributed Training - TinyML]], [[Parallelism That Splits Tensors Inside Models]], [[Pipeline Parallel]])?

Also inference is becoming more and more important and consumes main computing resources now. Hardware aware programming and optimization is taking more and more importance. There are still a lot of treasure to be discovered in the area of modern LLM.
