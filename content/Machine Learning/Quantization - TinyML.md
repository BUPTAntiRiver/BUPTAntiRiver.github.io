We can reduce calculation by reducing the neurons which directly reduces the computation numbers. Well, we can also reduce calculation time by reducing the time we need to finish a single process of computation by quantization.

# Definition

## Quantization

**Quantization** means we reduce the accuracy of the calculation, for example, we compute float rather than double or integer rather than float. Reducing the number of bits that represent each data can save the energy needed to compute.
**Quantization** is the process of constraining an input from a continuous or otherwise large set of values to a discrete one.

## Float-Point-Number

To represent a float point number we have three parts in bits, which are sign, exponential and fraction. Sign has only 1 bit, and exponential width decides range, fraction width decides precision.

# Quantization methods

## K-Means-Based Weight Quantization

Cluster the weight with nearest values and assign them with the same centroid value. Then we can use integer indices and float centroids to reduce the storage cost.
![[k-means-based weight quantization.png]]

### Fine-tuning quantized weights

Use the same quantization method for the weight error matrix which is the difference between centroids with the true weights and then apply the error to the original quantized weights.

## Linear Quantization

An affine mapping of integers to real numbers.
$r = (q - Z) \times S$

- $r$ is Floating-point
- $q$ is integer
- $Z$ is integer scale
- $S$ is Floating-point scale
  $S = \dfrac{r_{\max}-q_{\max}}{r_{\min}-q_{\min}}, Z=q_{\min} - \dfrac{r_{\min}}{S}$
  The calculation of $Z$ will need to take a `round()`.

### Quantization Granularity

With different quantization granularity, we will have different amounts of Floating-point scalars.

- Per-Tensor quantization
- Per-Channel quantization
- Group quantization

### Range Clipping

## Binary/Ternary Quantization

Quantize the weights to binary value $1$ and $-1$.

## Mixed Precision Quantization

Using reinforcement learning to help automatic quantization.
