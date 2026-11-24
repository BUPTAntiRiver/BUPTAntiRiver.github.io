# Introduction

In the context of image processing, we have two main ways to represent images: the **spatial** domain and **frequency** domain. We are more familiar with spatial domain (where images are represented as pixels), while the frequency domain offers powerful techniques for analyzing, filtering, compressing, and enhancing images.

## Spatial Domain

In the spatial domain, an image is represented by pixel intensities or brightness values at each position, like what we do in the famous hand written numbers recognition machine learning task. The MNIST dataset represents images in this way. This representation is intuitive for human, as each pixel directly corresponds to a point in the picture.

## Frequency Domain

In the frequency domain, the image is represented by its **frequency components**, which describes *how pixel values change across space*. This transformation is achieved using a **Fourier Transform**.

![[Image in Frequency Domain.png]]

Though frequency domain images are less intuitive to interpret, understanding them reveals critical information that we can’t easily access in the spatial domain.

# Differences: with filters as an example

There are both filters in spatial domain and frequency domain, which are both vital but work differently.

## Spatial Domain Filters

Apply convolution kernels ([[Convolution]]) directly to spatial domain images, for example: **low-pass** (Gaussian) and **high-pass** (Sobel). A Gaussian filter smooths the image by reducing fine details, similar to a low-pass filter in the frequency domain.

Spatial filtering is better for operations that focus on **localized regions** of the image like edge detection, smoothing or sharpening. For ***small*** kernels, spatial domain filtering is more efficient than frequency domain filtering because it ***avoids the overhead of Fourier transforms***.

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

# Resource Link

[Spatial Vs Frequency Domain: A Guide to Image Interpretation](https://medium.com/@chawthirisan/spatial-vs-frequency-domain-a-guide-to-image-interpretation-d9c16b129b3f)