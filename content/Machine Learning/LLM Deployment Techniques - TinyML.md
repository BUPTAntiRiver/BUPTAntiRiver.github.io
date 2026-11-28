# Quantization

## Activation Smoothing

The activation parameters differs a lot while weight parameters are closer to each other, so the former one is harder to quantize and the later one is easier. But we can multiply and divide activation and weight with the same scalar so that the result won't be change while making activation smoother, then we can let the quantization be easier.

## Weight-only quantization

W8A8 is not good enough, we might need W4A16

### AWQ (Activation-aware Weight Quantization)

## Smooth Attention

# Pruning and Sparsity

## Weight Sparsity (Pruning)

## Contextual Sparsity

example: Deja Vu

## Mixture-of-Experts (MoE)

Sparsely activated for each token

---

This part talks about a lot of tiny details in LLM deployment techniques, just look at the slide is probably much better.
