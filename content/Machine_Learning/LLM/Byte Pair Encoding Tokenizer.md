Tokenization takes an important part in large language models. A lot of problems come from it, but we cannot avoid it.

Tokenization is about how to transform the words or sequences that is understandable to human into a form that is easier to be managed with machine, like from a string to an array.

An simple way is we set an index for every character, that would be easy if only for English, but when considering other languages like Chinese, the vocabulary will become super large.

The modern method is usually byte level tokenization. We only need 256 index for all representation, actually, index is just the value itself. But the problem is this will make _sequence very long_, the original sequence with length 1000 may become 1500. And in Transformer complexity grows quadratically with sequence length.

So we use byte pair encoding tokenizer.

## What is new?

We consider pair of bytes, some specific byte pairs may occur more often, so we group them together and assign them a new index to represent them together. After we update the vocab, we also update the original text (in index form), we merge pair indices and do this repeatedly, until we reach the desiring vocabulary size.

## How to train?

This is where the demons come out. We are operating with terabytes of data, or even more. It would be very slow if we just update the whole text to do one merge every time.

### Pre-tokenization

We can break the text into coarse tokens first like word-level, and track their count, so that we can train the tokenizer base on these unique words which will be much faster.

### Special tokens

There are pre-defined special tokens like `end-of-text` or `start-of-text` to provide extra information. They are also raw bytes! But we should not train the tokenizer on them, or we will break it. So that when doing pre-tokenization, instead of building a separating pattern, we also need to protect the special tokens and neglect them when building the count table.

### Merging

This becomes the simplest part, just count and update. But how to implement it efficiently? Imagine a naive implementation: we scan all pretokens and compute the count, then update merges and vocab, then we need to apply the merge back to the pretokens so that we can do further merges with merged bytes. That will also need a scan through.

### Parallel Pre-tokenization

The biggest bottleneck of bpe training is the pre-tokenization. And luckily, we can do it in parallel.

## How to use?

### Encoding

The encoding of process of BPE mirrors how we train the BPE vocabulary. The major steps are:

1. **Pre-tokenize**. We also pre-tokenize the sequence and encode each pre-token as a sequence of UTF-8 bytes, then we will merge these bytes within each pre-token into vocabulary elements.
2. **Apply the merges**. Take the sequence of vocabulary element merges created during BPE training, then apply it to our pre-tokens _in the same order of creation_.

Also we should handle user-defined specially tokens properly during encoding. Memory constraints is also important, when dealing with large files we can chunk it, but should make sure no tokens crosses chunk boundaries.

Encoding is just like merging, which is time consuming. So we may apply some optimizations:

1. **Cache**. Some words are very common in text like: the, a. We can cache the most common 1000 words, then in most of the time the tokenizer hits the cache.
2. **Rank-based Lookups**: instead of checking the merges one by one, we can build a dict that maps merge to its priority rank, so that we can just scan through the word to know which merge should be applied.

### Decoding

Decoding is much simpler, we can simply look up the vocabulary and concatenate them together. However the user input is not guaranteed to map to valid Unicode strings (we are playing with bytes), in this case we should replace the malformed bytes with the official Unicode replacement character `U+FFFD`. We can do this by setting the error flag of `bytes.decode` to `errors='replace'`.
