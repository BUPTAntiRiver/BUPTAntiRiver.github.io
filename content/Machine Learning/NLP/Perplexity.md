[Perplexity for LLM Evaluation - GeeksforGeeks](https://www.geeksforgeeks.org/nlp/perplexity-for-llm-evaluation/)
**Perplexity** is a **metric** which measures the **uncertainty** of a model's **prediction**.
$$
\text{Perplexity}=2^{H(p)}
$$
$H(p)$ stands for entropy of the model's predictions.
If the model is very certain about the prediction, then entropy would be small and perplexity would be lower, so the model performs better and has higher confidence.
This makes sense, imagine a scenario, a teacher is asking a student, we want the student to answer both correctly and confidently without hesitation.