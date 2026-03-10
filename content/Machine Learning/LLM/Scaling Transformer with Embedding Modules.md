---
aliases:
  - STEM
---

Paper link: http://arxiv.org/abs/2601.10639

This paper is said to explore architecture in large language models even earlier than [[Conditional Memory via Scalable Lookup - Engram|DeepSeek Engram]]. Also I wants to make use of the paper reading method I have learned from a paper.

# First Pass

This paper is mainly about how to improve current system performance on _popular_ sparse models. Sparse models means, though the parameter capacity or you can say model size is very big, but the activated parameters during computation are only part of them, which could be rather small, like [[MoE-Mixture-of-Experts|MoE]]. This method is _popular_ is because it can increase model size but not increase computation cost.

The problem of current sparse models is that, sparse computation might not fully utilize compute power, lack parameter access locality, degrade kernel efficiency and raise the number of all-to-all message communication, yielding suboptimal end-to-end performance.
