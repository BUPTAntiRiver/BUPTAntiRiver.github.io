http://arxiv.org/abs/2510.18234
This is an initial investigation into the feasibility of compressing long context via optical 2D mapping. DeepSeek-OCR consists of two components: DeepEncoder and DeepSeek3B-MoE-A570M as the decoder. It can achieve high decoding precision with pretty high compression ratio.

# Introduction

Current Large Language Models face significant computational challenges when processing long textual context due to quadratic scaling with sequence length. They explore a potential solution: leveraging visual modality as an efficient compression medium for textual information.
Their work has three main contributions:

1. Comprehensive analysis on visual-text token compression ratios.
2. DeepEncoder, a novel architecture.
3. DeepSeek-OCR, based on DeepEncoder and DeepSeek3B-MoE-A570M.

# Related Works

## Typical Vision Encoders in VLMs

This part mainly tells us that when dealing with high resolution images, the existing vision encoders faces challenges like: problem with pipeline parallelism; low native encoder resolution to reduce activation memory but result in too much vision tokens; massive activation memory that can cause GPU overflow, and extremely long sequence length that slow down prefill and generation phase of inference.

## End-to-end OCR Models

This is quite mature, so inspired them to try DeepSeek-OCR.

# Methodology

## Architecture

The encoder is mainly composed of an 80M SAM-base and a 300M CLIP-large connected in series. The decoder adopts a 3B MoE architecture with 570M activated parameters. See the paper graph for more detail.

## DeepEncoder

The vision encoder we need should have these following features:

1. Capable of processing high resolutions;
2. Low activation at high resolutions;
3. Few vision tokens;
4. Support for multiple resolution inputs;
5. Moderate parameter count.

### Architecture of DeepEncoder

Consists of 2 components: a visual _perception feature extraction_ component dominated by window attention, and a visual _knowledge feature extraction_ component with dense global attention. They use _SAM-base_ (patch-size 16) and _CLIP-large_ as the main architectures for the 2 components respectively. For CLIP, they remove the first patch embedding layer since the input is no longer images but output tokens from the previous pipeline. Between the two components, they borrow form Vary use a 2-layer convolution network to perform 16$\times$ downsampling of vision tokens.

### Multiple Resolution Support

It has four **native** resolution supports and **dynamic** resolution supports by combing the native ones. The native ones are done through resizing or padding.
Native means, in this mode, the model process the whole input picture at once by resizing or padding it into demanding resolution shape, then generate corresponding amount of visual tokens.
Dynamic means, the model separate the input picture into smaller pieces and process them in Tiny or Small mode (two modes in native) and generate a whole overview in Base or Large mode (also two modes in native).
These different modes are trained together, besides Gundam-master mode, because it is too large so it is trained alone afterwards.

## The MoE Decoder

Their decoder uses DeepSeekMoE, specifically DeepSeek-3B-MoE. The decoder reconstructs the original text representation from the compressed latent vision tokens of DeepEncoder as:

$$
f_{\text{dec}}:\mathbb{R}^{n \times d_{\text{latent}}} \to \mathbb{R}^{N\times d_{\text{text}}};\hat{\mathbf{X}}=f_{\text{dec}}(\mathbf{Z})\quad \text{where }n\leq N
$$

## Data Engine

They construct complex and diverse training data. OCR 1.0 and 2.0 are the main data, they also produced general vision data and text-only data. The data construction detail are in the paper.

### OCR 1.0 data

1.0 mainly consists of traditional OCR tasks such as scene image OCR and document OCR.
Document data is the top priority for DeepSeek-OCR.

### OCR 2.0 data

2.0 mainly includes parsing tasks for complex artificial images.

## Training Pipelines

This phase is very simple and mainly consists of two stages: a).Training DeepEncoder independently; b).Training the DeepSeek-OCR. Note that the Gundam-master mode is obtained by continuing training on a pre-trained DeepSeek-OCR model with 6M sampled data.
See the paper for more detail.

# Conclusion

This paper proposes a very impressive point that is we can compress long context information by compressing the image of the context words. They reached nearly loss-less 10 times compression, and explored substantial new room for research and improvement.
And this kind of image compression is quite similar to human, because when we are reading words, we are actually processing the image our eyes perceived!
