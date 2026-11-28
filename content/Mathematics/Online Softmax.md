Given a tensor of logits $\mathbf{x}$, softmax is a normalization method. Usually $\mathbf{x}$ has a batch dimension for parallel process. It works like this:

$$
	\text{Softmax}(\mathbf{x}) = \frac{\exp(\mathbf{x})}{\sum \exp(\mathbf{x})}
$$

The $\exp$ in denominator and the div is element-wise operation.

Since $\exp$ grows exponentially, for numerical stability we subtract $\max(\mathbf{x})$ for each element, which does not affect final result. So it becomes:

$$
	\text{Softmax}(\mathbf{x}) = \frac{\exp(\mathbf{x}-\max(\mathbf{x}))}{\sum \exp(\mathbf{x}-\max(\mathbf{x}))}
$$

Now there is another problem, memory space limit. For huge tensor with shape `(B, L)`, the processor need to handle `(batch_size, L)` to do softmax operation since it requires a global max and sum. And in modern long context scenario, `L` will be a large number which may lead to insufficient processor memory space.

To solve this problem, we need to divide `L` dimension into pieces too. So maintaining a running max and sum will be the key point.

The pseudo code:

```python
# input and output stands for given in and out tensor
for b_start in range(0, B, B_BLOCK_SIZE):
	b_end = b_start + B_BLOCK_SIZE
	s = allocate(B_BLOCK_SIZE, 1) # init with 0
	curr_max = allocate(B_BLOCK_SIZE, 1)
	prev_max = allocate(B_BLOCK_SIZE, 1) # init with -inf
	for l_start in range(0, L, L_BLOCK_SIZE):
		l_end = l_start + L_BLOCK_SIZE
		x = input[b_start:b_end][l_start:l_end]
		curr_max = max(x, axis=-1, keepdims=True)
		curr_max = max(curr_max, prev_max)
		o = exp(x - curr_max) # a broadcast here
		s = sum(o) + s * exp(prev_max - curr_max) # a running updating sum
		prev_max = curr_max

	for l_start in range(0, L, L_BLOCK_SIZE):
		l_end = l_start + L_BLOCK_SIZE
		x = input[b_start:b_end][l_start:l_end]
		o = exp(x - prev_max) # now prev_max is global max
		output[b_start:b_end][l_start:l_end] = div(o, s)
```

I evicted a lot of details here like mask, shape, whatever, but the semantic is that we keep recording previous max, and update the sum correctly by removing old max and applying new max, which is that `s * exp(prev_max - curr_max)` part.

Such technique is also applied in [[FlashAttention]].