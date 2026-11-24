# Introduction

In the context of image processing, we have two main ways to represent images: the **spatial** domain and **frequency** domain. We are more familiar with spatial domain (where images are represented as pixels), while the frequency domain offers powerful techniques for analyzing, filtering, compressing, and enhancing images.

## Spatial Domain

In the spatial domain, an image is represented by pixel intensities or brightness values at each position, like what we do in the famous hand written numbers recognition machine learning task. The MNIST dataset represents images in this way. This representation is intuitive for human, as each pixel directly corresponds to a point in the picture.

## Frequency Domain

In the frequency domain, the image is represented by its **frequency components**, which describes _how pixel values change across space_. This transformation is achieved using a **Fourier Transform**.

![[Image in Frequency Domain.png]]

Though frequency domain images are less intuitive to interpret, understanding them reveals critical information that we can’t easily access in the spatial domain.

# Differences: with filters as an example

There are both filters in spatial domain and frequency domain, which are both vital but work differently.

## Spatial Domain Filters

Apply convolution kernels ([[Convolution]]) directly to spatial domain images, for example: **low-pass** (Gaussian) and **high-pass** (Sobel). A Gaussian filter smooths the image by reducing fine details, similar to a low-pass filter in the frequency domain.

Spatial filtering is better for operations that focus on **localized regions** of the image like edge detection, smoothing or sharpening. For **_small_** kernels, spatial domain filtering is more efficient than frequency domain filtering because it **_avoids the overhead of Fourier transforms_**.

## Frequency Domain Filters

Apply **frequency masks**. These masks can block or enhance specific frequency ranges in the Fourier-transformed image. A frequency domain low-pass filter would retain only low frequencies, smoothing out rapid changes (like edges), similar to a spatial domain Gaussian filter.

Large kernel in spatial domain can be computationally expensive, while in frequency domain, convolution becomes multiplication, so frequency domain filtering **excels** at addressing **global image properties** like **removing periodic noise or patterns or textures**, enhancing edges at specific scales, or filtering out specific frequency band.

# Usage: with Convolution Neural Networks as an example

Why we only have CNN which uses spatial domain filter but don't have a FNN that uses frequency masks on images? There are several theoretical and practical reasons, which helps us understand the difference between spatial and frequency domain better:

1. The design target of CNN is to **extract localized feature** and extract various level of them through those layers, which meets the advantage of spatial domain.
2. Efficiency matters, abandon unnecessary Fourier transformation overhead.
3. Data augmentation compatibility. In CNN we flip, rotate, pad images to make model robust, but that would require a Fourier transform each time we do the modification to image, which is costly.
4. Frequency filtering is great for periodic patterns and textures but struggles with non-periodic and irregular patterns, which are common in natural images. CNN can handle both cases with its deep layers.

However, there are emerging frequency-based techniques like Fourier CNN and hybrid approaches. There are still a lot to explore, since the philosophy of computer science is usually find two opposite extreme and combine them to reach the best.

# Practice

The figure above pairs the original image with its **log-magnitude spectrum**. Reproducing that pair step by step makes the abstract claims above concrete.

## Step 1: Collapse to a Single Channel

The transform is defined on a real-valued scalar field, so an RGB image must first be reduced to one brightness channel. The standard choice is luma, with the ITU-R BT.601 weights:

$$
I(x,y) = 0.299\,R(x,y) + 0.587\,G(x,y) + 0.114\,B(x,y)
$$

It weights the channels by their perceptual contribution instead of averaging them. Transforming each channel separately and recombining is also valid, but contributes nothing to the visualization, so we skip it.

## Step 2: Apply the 2D Discrete Fourier Transform

For an $M \times N$ image, the DFT is

$$
F(u,v) = \sum_{x=0}^{M-1}\sum_{y=0}^{N-1} I(x,y)\, e^{-2\pi i\left(\frac{ux}{M} + \frac{vy}{N}\right)},\qquad u \in [0, M-1],\ v \in [0, N-1]
$$

Each coefficient $F(u,v)$ is **complex**, so it carries two pieces of information:

- the **magnitude** $|F(u,v)|$, which says how much of that frequency is present;
- the **phase** $\arg F(u,v)$, which says where in space that frequency component sits.

Here $u$ counts cycles across the width and $v$ cycles down the height. The index $u \in [0, M/2]$ therefore encodes the positive frequency $u/M$ cycles per pixel, up to the Nyquist limit of $0.5$; indices above $M/2$ wrap around to the corresponding negative frequencies.

The right panel keeps the magnitude and discards the phase. That is precisely why the spectrum cannot be inverted back into the photograph.

## Step 3: Compress the Dynamic Range

The zero-frequency coefficient is the sum of all pixel values, $F(0,0) = MN\bar{I}$ where $\bar{I}$ is the mean brightness, so it is orders of magnitude larger than any other coefficient. Plotted on a linear scale, the spectrum collapses into a single bright dot on black. We therefore display

$$
D(u,v) = \log\!\left(1 + |F(u,v)|\right)
$$

normalized to $[0,1]$ and then quantized to 8 bits. The $+1$ keeps the log finite and maps $|F| = 0$ to $0$. This is a display choice only: the transform data is untouched, and the compression is what makes the faint high-frequency structure visible at all.

## Step 4: Move the Zero Frequency to the Center

By default the DFT stores $u = v = 0$ at index $(0,0)$, so the spectrum appears split across the four corners. Rolling the array by $(M/2, N/2)$ (`np.fft.fftshift` in NumPy, `fftshift` in MATLAB) puts the zero frequency in the middle, which matches how we read a plot: low frequencies at the center, high frequencies toward the edges. The same shift follows from the shift theorem by multiplying the image with $(-1)^{x+y}$, provided $M$ and $N$ are even. Either way it is a display convention and changes no information.

## Step 5: Read the Spectrum

With the spectrum centered, the figure becomes legible:

| What you see                        | What it means                                                                                    |
| ----------------------------------- | ------------------------------------------------------------------------------------------------ |
| Bright point at the center          | The DC term, i.e. the mean brightness of the whole image                                         |
| Brightness falling off with radius  | Natural images carry most of their energy in low frequencies                                     |
| Faint grain near the edges          | High-frequency content: grass texture, film grain, noise                                         |
| Straight streaks through the center | Dominant straight edges in the scene, each producing a line perpendicular to its own orientation |

The tripod legs, roof lines and horizon in the original each leave one of those streaks. Two properties also follow directly from the algebra. Because $I$ is real, $F(-u,-v) = \overline{F(u,v)}$, so the magnitude spectrum is symmetric under a $180^\circ$ rotation; the two halves are not independent. And because distance from the center measures frequency, the edge midpoints of the plot sit at Nyquist ($0.5$ cycles per pixel) and the corners at $\sqrt{2}/2 \approx 0.707$, so the grain out there is structure at the scale of individual pixels.

## Reference Implementation

```python
import numpy as np
from PIL import Image

# Collapse to one channel, scaled to [0, 1].
image = np.asarray(Image.open("cameraman.png").convert("L"), dtype=np.float64) / 255.0

# Complex spectrum; the DC term still sits at index (0, 0).
spectrum = np.fft.fft2(image)

# Center the spectrum and keep the magnitude.
magnitude = np.abs(np.fft.fftshift(spectrum))

# Log compression, then normalize to 8 bits for display.
display = np.log1p(magnitude)
display = (display - display.min()) / (display.max() - display.min())
Image.fromarray((display * 255).astype(np.uint8)).save("spectrum.png")
```

`fft2` returns the unnormalized sum, so the absolute scale depends on $M$ and $N$; for display this constant factor is irrelevant. The phase is available as `np.angle(spectrum)` if you want to inspect it.

## What the Exercise Shows

- **The transform is global.** Every coefficient $F(u,v)$ is a weighted sum over every pixel, while a convolution kernel only ever sees a local neighbourhood. Frequency filtering is therefore naturally suited to image-wide properties, and it is the counterpart to the locality argument made earlier.
- **It is worth the overhead only for larger kernels.** A direct $k \times k$ convolution costs $O(MNk^2)$, while the transform route costs $O(MN\log MN)$; up to constant factors the two meet when $k^2 \approx \log_2(MN)$. Small kernels are cheaper in the spatial domain, large ones in the frequency domain.
- **Nothing is lost by transforming.** The phase is hidden in the figure, not discarded by the computation: `np.fft.ifft2(spectrum).real` returns the original image to within floating-point rounding. Filtering is then a pointwise multiplication of the spectrum by a mask plus one inverse transform.
- **Multiplying spectra is circular convolution.** If you want a linear convolution, pad the image before transforming, otherwise content from one edge wraps around to the opposite edge.

# Resource Link

[Spatial Vs Frequency Domain: A Guide to Image Interpretation](https://medium.com/@chawthirisan/spatial-vs-frequency-domain-a-guide-to-image-interpretation-d9c16b129b3f)
