`MinHash` is a method used in data deduplication. It plays like an estimate to [[Jaccard Similarity]], that tells how similar two sets are. In language model case, we are detecting how similar two pieces of text are.

# Some Work Before MinHash

First, we need to turn the texts into a set of tokens.

# What does MinHash do?

## Algorithm

In MinHash, we have a bunch of hash functions, and we are going to apply them to all the tokenized texts, which are just sets now.

For each hash function, we can consider them as a _permutation_, that turns tokens into some kind of id. Since then, if two sets have the same smallest id under same hash function, then it tells they must have one shared element, which contributes to the numerator of Jaccard Similarity.

With the results of multiple hash functions, we can say that:

$$
\text{Pr}[h_{\min}(A)=h_{\min}(B)]=J(A,B)
$$

## LSH (Locality-sensitive Hashing)

MinHash is a kind of LSH, we cannot compute the Jaccard for every pair of texts, so we consider MinHash probability as a approximation and take is as a coarse first step.

With the MinHash score we have got, we can fill similar piece of texts into the same bucket, and add the pairs in same bucket to the candidates list.

Finally compute the true Jaccard on the candidates.
