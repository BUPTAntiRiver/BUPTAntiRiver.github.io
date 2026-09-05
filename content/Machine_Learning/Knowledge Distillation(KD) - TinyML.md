# What is Knowledge Distillation?

**Knowledge distillation** is a model compression technique where a large, well-performing model (**teacher**) is used to train a smaller, more efficient model (**student**) by transferring its "knowledge" — not just hard labels, but also soft output distributions.

- The student learns from both:
  - The **true labels** (as in normal supervised learning)
  - The **teacher’s soft predictions**, which contain additional structure or “dark knowledge”

# Why use Knowledge Distillation?

To create **smaller, faster models** that retain **high accuracy** — ideal for deployment on **mobile, embedded, or edge devices (TinyML)**.

- Training a small model directly may result in **lower accuracy**.
- Distillation helps the student learn **not only what is right**, but also **how wrong classes relate to the right one**, improving generalization.

Common use cases:

- Deploying BERT → DistilBERT
- Compressing ResNet models for mobile
- Combining ensemble models into one lightweight student

# How does Knowledge Distillation work?

### Step-by-step:

1. **Train the teacher model** normally (e.g., a large CNN or transformer).
2. **Design a smaller student model** that you want to deploy.
3. **Train the student** using a combination of:
   - Standard cross-entropy loss with the true labels
   - KL divergence (or cross-entropy) between **SoftMax outputs** of teacher and student (softened using a temperature $T > 1$)

### Distillation loss formula:

$$\text{Loss} = \alpha \cdot \text{KL}(S_T, T_T) + (1 - \alpha) \cdot \text{CE}(S, y)$$
Where:

- $S$, $T$: student and teacher logits
- $T_T$: softened teacher output: $\text{Softmax}(T / T)$
- $S_T$: softened student output: $\text{Softmax}(S / T)$
- $y$: true label
- $\alpha$: weight between distillation and supervised loss
