## Tensor Memory

When learning programming languages, we know that we have at least two methods to store floating point values: `float`, `double`. `float` takes 32 bits, which is 4 bytes and `double` takes 64 bits, which is 8 bytes.
The bits consist of one _sign_ bit, some _exponent_ bits and some _fraction_ bits. You may ask AI for more detail.
We usually use `float32` (32 bits) in deep learning, it is precise enough. But to save memory, we also produced `float16` and `bfloat16`, the former has better precision and the latter has larger range.
There are some bald choices like `float8`, which is more risky and unstable. So, in practice, mixed precision training is more common. e.g. we may use `float32` on accumulating operations like reduce or attention, while `bfloat16` on simple run through computations like feed forward.

## Compute Accounting

### Tensor Einops

[Einops Tutorial](https://einops.rocks/) Better way to modify your tensor rather than using `-2, -1` indices and comments, it makes every thing clearer.

### Tensor Operations FLOPs

A floating-point operation (FLOP) is a basic operation like addition or multiplication.
In one pass of simple MLP the total FLOPs is: 6\*(\#data points)(\#parameters), 2 for forward, 4 for backward. The 2 stands for 1 addition and 1 multiplication, because in linear layer we have $y=\sum_{i=1}^{n}w_{i}x_{i}$. Backward is two times of forward is because we also need to do addition and multiplication for gradients.

## Models

### Initialization

Instead of just using random number, some predefined initialization function might make the output more stable.
Note about randomness: to be able to reproduce, setting a random seed manually is a good choice.
