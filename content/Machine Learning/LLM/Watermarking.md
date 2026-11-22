# Introduction

Text watermarking is an old topic, we suppose we have the original text $t$, watermark information $w$, watermark algorithm $F$, watermarked text $t_{w}=F(t, w)$, and detect algorithm $D(t_{w})=w$ which can extract watermark information back.

A good watermark can be retrieved clearly and be robust to watermark removal. Also it shouldn't change the semantic meaning of original text too much, which may cause loss of original information and leave an obvious watermarked signal to the reader.

There are a lot of interesting methods to watermark text in the past, but we won't talk about them here, we focus on the newly emerged methods targeting large language model generated text.

Intuitively, these methods work on processes that are only owned by large language models, which are **Model Training, Logits Generation and Token Sampling**.

# How?

We present a model for embedding watermarks in generated content: $\text{Embed}(\mathbf{p},w)\to \mathbf{z}, \text{Generate}(\mathbf{z},\Theta)\to \mathbf{y}$, where $\mathbf{p}$ is prompt, $w$ is watermark, $\mathbf{z}$ is watermarked representation, $\Theta$ is model parameters and $\mathbf{y}$ is final output.

## Training Phase

Training phase watermarking embeds a unique identifier $w$ into an LLM by modifying a subset of the training data. Consider an LLM trained on 100 M web-text sentence. To encode a 8-bit string $w=11001000$, we first sample 5% of the training corpus to form $D_{s}$. Two low frequency yet grammatical tokens like "*amber-fern*" (bit 0) and "*azure-reed*" (bit 1) serve as markers. Traversing the bit string left to right, we append the corresponding marker to each sentence in $D_{s}$, producing the watermarked dataset $D_{m}$.

Next, we merge $D_{m}$ with original dataset using a weighting factor $\alpha$ to obtain the final training set $D_{w}$:

$$
	D_{w} = \alpha \cdot (D\setminus D_{s}) \cup (1-\alpha) \cdot D_{m}.
$$

An LLM trained on $D_{w}$ becomes $L_{w}$; when prompted with text similar to $D_{m}$, $L_{w}$ reproduces the marker sequence, thereby revealing $w$.

## Logits Generation

Watermarking at the logits generation phase embeds a message $w$ by perturbing the logits without touching the model's weights. Assume we wish to hide the 3-bit string $w=101$ in a short reply. Using the KGW method as an example, we first compute for each step $i$, a green vocabulary $G_{i}\in V$ such that roughly $\gamma=0.5$ of the tokens are eligible. When ever the next bit equals 1, we add a small positive bias $\beta$ to every logit in $G_{i}$; when the bit equals 0, all the logits are left unchanged. A detector can later verify that the sequence contains significantly more green tokens than chance would predict, thereby recovering the embedded bits.

This raw implementation might change semantic and the watermark can be hard to extract, so there are more studies have been done to improve these perspectives like: **Semantic Robustness, Optimal, Imperceptibility, Diversity**.

## Token Sampling

Token sampling watermarking is very similar to logits generation watermarking in my opinion, because the probability during sampling is calculated base on logits. But there are still slight differences, since we modify probabilities in this phase but not logits.

In general, a watermark aware sampler uses $\mathbf{o}_{i}=\Sigma_{w}(\text{softmax}(\mathbf{z}^{(i)}),w)$, so that each choice is slightly nudged toward or away from vocabulary indices that represent $w$. One simple implementation scales every probability $p_{j}$ through a weighted function $\delta(w,j)$:

$$
	\Sigma _{w}(p, w) = \{p_{j}\delta(w,j)|j=1,2,\dots,N\},
$$

where $N$ is vocabulary size. The shape of $\delta$ governs the trade-off between watermark strength and readability. Some recent works refine the scaling step, converting it from a static process into an adaptive decision mechanism. Like only trigger watermark when the predicted distribution is sufficiently entropic.

# Resources

## Surveys and reviews

- **[A Survey of Text Watermarking in the Era of Large Language Models](https://arxiv.org/abs/2312.07913)** — Li et al., 2023. Taxonomy of watermarking for LLMs and for LLM-generated text, plus attacks and evaluation.
- **[Watermarking Techniques for Large Language Models: A Survey](https://arxiv.org/abs/2409.00089)** — 2024. Method-by-method overview, shorter and more up to date.
- **[LLM Watermarking as Big Data Provenance: A Deployment-Oriented Systematization](https://arxiv.org/abs/2607.10103)** — 2026. Deployment requirements: serving cost, population-scale false positives, multi-key attribution, long-term verification.
- **[Are LLM watermarks reliable in practice? A systematic review](https://www.sciencedirect.com/science/article/pii/S1574013726001826)** — 2026. Evidence, robustness, threat models, and deployment assumptions.
- **[Securing Large Language Models: A Survey of Watermarking and Fingerprinting Techniques](https://doi.org/10.1145/3773028)** — ACM Computing Surveys. Situates watermarking next to fingerprinting and provenance.

## Core methods

- **[A Watermark for Large Language Models](https://arxiv.org/abs/2301.10226)** — Kirchenbauer et al., ICML 2023. Green-list logit bias with a keyed vocabulary partition, detected by a one-sided z-test.
- **[On the Reliability of Watermarks for Large Language Models](https://arxiv.org/abs/2306.04634)** — 2023. Detection under edits, translation, and long documents; tuning the watermark strength.
- **[Provable Robust Watermarking for AI-Generated Text](https://arxiv.org/abs/2306.17439)** — 2023. Gumbel watermark: keyed Gumbel-max sampling with provable detection and edit robustness.
- **[Robust Distortion-free Watermarks for Language Models](https://arxiv.org/abs/2307.15593)** — Kuditipudi et al., 2023. Inverse-transform sampling against a keyed uniform sequence with edit-distance alignment at detection.
- **[Unbiased Watermark for Large Language Models](https://arxiv.org/abs/2310.10669)** — 2023. Reweights the next-token distribution so the watermark is unbiased in expectation yet still detectable.
- **[A Semantic Invariant Robust Watermark for Large Language Models](https://arxiv.org/abs/2310.06356)** — 2023. Signal carried at the semantic level to survive paraphrasing.
- **[Scalable watermarking for identifying large language model outputs](https://www.nature.com/articles/s41586-024-08025-4)** — Dathathri et al., Nature 2024. SynthID-Text: tournament sampling with keyed _g_-values, deployed in Gemini, detected without model access.
- **[Three Bricks to Consolidate Watermarks for Large Language Models](https://arxiv.org/abs/2308.00113)** — Fernandez et al., 2023. WANDA, combining several weak signals into a stronger detector with honest quality/robustness curves.
- **[PostMark: A Robust Blackbox Watermark for Large Language Models](https://arxiv.org/abs/2406.14517)** — 2024. Post-processing watermark for providers without logits access.
- **[Towards Codable Watermarking for Injecting Multi-bits Information to LLMs](https://arxiv.org/abs/2307.15992)** — 2023. Multi-bit payloads via constrained decoding over balanced green/red lists.
- **[REMARK-LLM: A Robust and Efficient Watermarking Framework for Generative Large Language Models](https://arxiv.org/abs/2310.12362)** — 2023. Learned, robust detector rather than a fixed statistical test.

## Theory

- **[Undetectable Watermarks for Language Models](https://arxiv.org/abs/2306.09194)** — Christ, Gunn, Zamir, 2023. Formal undetectability from cryptographic PRFs against adversaries with sampling access.
- **[Watermarks in the Sand: Impossibility of Strong Watermarking for Generative Models](https://arxiv.org/abs/2311.04378)** — Zhang et al., 2023. Impossibility against an adversary who can query the detector and paraphrase.
- **[Can AI-Generated Text be Reliably Detected?](https://arxiv.org/abs/2303.11156)** — Sadasivan et al., 2023. Upper bound on achievable detection under recursive paraphrasing.
- **[Could LLM Watermark Detection be Public?](https://arxiv.org/abs/2610.12106)** — 2026. Quantifies the risk of releasing detectors at several levels of access.
- **[Watermarkability: Measuring and Improving Language-Model Compatibility with Logit-Based Watermarks](https://arxiv.org/abs/2610.22810)** — 2026. Defines a metric for how readily a model accepts logit-based watermarks.
- **[PVMark: Enabling Public Verifiability for LLM Watermarking Schemes](https://arxiv.org/abs/2510.26274)** — 2025. Verification without the secret key.

## Attacks and robustness

- **[Watermark Stealing in Large Language Models](https://arxiv.org/abs/2402.19361)** — Jovanović et al., ICML 2024. Recovers the keyed rule through API access, then forges or strips marks.
- **[Paraphrasing evades detectors of AI-generated text, but retrieval is an effective defense](https://arxiv.org/abs/2303.13408)** — Krishna et al., 2023. DIPPER paraphrase attack.
- **[A Unified Attack Framework against Statistical Generative Watermarks for Large Language Models](https://arxiv.org/abs/2610.27653)** — 2026. Shared weakness across KGW, GumbelSoft, DipMark, ITS, and SynthID.
- **[Blind PRNG Hijacking: An Undetectable Integrity-Preserving Attack Against LLM Watermarking](https://arxiv.org/abs/2605.28632)** — 2026. Attacks the keyed randomness while leaving text quality untouched.
- **[Watermark Basins: Black-Box Detection of Distilled LLMs](https://arxiv.org/abs/2610.37266)** — 2026. Teacher watermarks as "radioactivity" for detecting API distillation.

## Evaluation and tooling

- **[Mark My Words: Analyzing and Evaluating Language Model Watermarks](https://arxiv.org/abs/2312.00273)** — 2023. Common evaluation harness for quality, detectability, and robustness.
- **[MarkLLM: An Open-Source Toolkit for LLM Watermarking](https://arxiv.org/abs/2405.10051)** — Pan et al., EMNLP 2024 demo. Unified implementations, mechanism visualization, 12 evaluation tools.
- **[DetectGPT](https://arxiv.org/abs/2301.11305)** — Mitchell et al., 2023. Zero-shot detection baseline via probability curvature.
- **[Fast-DetectGPT](https://arxiv.org/abs/2310.05130)** — 2023. Faster zero-shot detection baseline.

## Recent work (2025–2026)

- **[Echoes within the Reasoning: Stealthy and Effective Watermarking via Chain of Thought](https://arxiv.org/abs/2605.28890)** — 2026. Watermarks reasoning traces with sentinel tokens and a logprob-based verifier.
- **[Session-Bound Behavioral Watermarking for Autonomous LLM Agents](https://arxiv.org/abs/2610.38579)** — 2026. Marks tied to agent sessions rather than text.
- **[Watermarking Diffusion Language Models with Learned Foresight and Balanced Bands](https://arxiv.org/abs/2610.32205)** — 2026. Watermarking diffusion LMs.
- **[RoCoMark: Robust Code Watermarking Against LLM-Based Functionality-Preserving Rewriting Attacks](https://arxiv.org/abs/2611.11399)** — 2026. Code watermarking under renaming and refactoring.
- **[Can Watermarking Techniques Help Prevent LLM Model Stealing?](https://arxiv.org/abs/2607.10794)** — 2026. Watermarking as a defense against extraction.
- **[Reconciling Privacy with Provenance for Language Model Watermarks](https://arxiv.org/abs/2611.06100)** — 2026. Tension between per-user attribution and privacy.
- **[Implicit Identity Technologies for LLMs: Fingerprinting and Watermarking across Datasets, Models, and Generated Content](https://arxiv.org/abs/2605.29245)** — 2026. Survey connecting dataset-, model-, and output-level identity.
- **[HiMark: Hierarchical and Controllable Multi-bit Watermarking for Large Language Models](https://arxiv.org/abs/2610.30601)** — 2026. Multi-bit watermarking with hierarchical control.

## Code

- **[THU-BPM/MarkLLM](https://github.com/THU-BPM/MarkLLM)** — Reference implementations of KGW, ITS, SIR, Gumbel, and SynthID-style schemes, plus visualization.
- **[jwkirchenbauer/lm-watermarking](https://github.com/jwkirchenbauer/lm-watermarking)** — Original KGW implementation.
- **[facebookresearch/three_bricks](https://github.com/facebookresearch/three_bricks)** — WANDA detector and evaluation scripts.
- **[google-deepmind/synthid-text](https://github.com/google-deepmind/synthid-text)** — Tournament sampling and detector.

## Standards, policy, and beyond text

- **[C2PA](https://c2pa.org)** — Signed content metadata (Content Credentials); provenance rather than a statistical watermark.
- **[EU AI Act, Article 50](https://artificialintelligenceact.eu/article/50/)** — Transparency obligations for AI-generated content.
- **[NIST AI 100-4](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-4.pdf)** — Synthetic content risk mitigation, with watermarking as one control.
- **[Tree-Ring Watermarks](https://arxiv.org/abs/2305.20030)** — Wen et al., 2023. Keyed fingerprints for diffusion images.
- **[Invisible Image Watermarks Are Provably Removable Using Generative AI](https://arxiv.org/abs/2306.01953)** — 2023. Removal attack on image watermarks.
