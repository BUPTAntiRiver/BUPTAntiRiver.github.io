# Links

https://en.wikipedia.org/wiki/Bloom_filter

# Algorithm

A bloom filter is a space-efficient probabilistic data structure that is used to tell whether an element is a member of a set. False matches are possible but false negatives are impossible.

An empty Bloom filter is a bit array of $m$ bits, all set to 0. It is equipped with $k$ different hash functions, which map set elements to one of the $m$ possible array positions.

To _add_ an element, feed it to all hash functions, and set all corresponding bits to 1. To _test_ whether an element is in the set, feed it to all hash functions, if any bit is 0 then the element is not in the set, otherwise, it might be in the set or all bits are accidentally set to 1 by other elements.

# Probability of false positive

Assume hash functions select each array position with equal probability. For $m$ bits and $k$ hash functions, the probability that one position is not set to 1 when inserting 1 element is $(1-\frac{1}{m})^{k}$. For large $m$, we have $\left( 1-\frac{1}{m} \right)^{k}\approx e^{-k/m}$.

If we insert $n$ elements, the probability that a certain bit is still 0 becomes $e^{-kn/m}$, then the prob it is 1 is $1-e^{-kn/m}$.

The false positive is the probability of $k$ results are all 1, which is $(1-e^{-kn/m})^{k}$. This is not strictly correct as it assumes independence of each bit being set. However this is a close approximation we have that the false positive probability decreases as $m$ increases and increases as $n$ increases.
