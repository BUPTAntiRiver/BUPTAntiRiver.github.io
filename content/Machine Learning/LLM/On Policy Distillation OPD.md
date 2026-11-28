On Policy Distillation (OPD) is a method that kind of lives between Supervised Fine-Tuning (SFT) and Reinforcement Learning (RL).

It is like Knowledge Distillation (KD) but not in the SFT way that let the _student model_ directly learns the generated logits of _teacher model_, which may have deviation because the student is just copying the result of teacher. In real life this is not considered as a good method.

The loss of KD is:

$$
\mathcal{L}=\alpha \mathcal{L}_{CE} + (1-\alpha)\mathcal{L}_{KD}
$$

where $\mathcal{L}_{CE}$ is the cross entropy loss between student and ground truth labels, and $\mathcal{L}_{KD}$ is the distillation loss between student and teacher outputs.

The KD loss is commonly:

$$
	\mathcal{L}_{KD}=T^{2}\cdot KL\left( \sigma\left( \frac{z_{t}}{T}\right)\|\sigma \left( \frac{z_{s}}{T} \right) \right)
$$

where $z$ are logits of teacher and student, $T$ is temperature, and $KL$ means KL divergence.

What is different of OPD is that it let students do the full rollout first, and use the teacher's result based on students' result as guiding signal, so it is more like teacher is correcting the students.

So its objective is a KL-divergence between teacher and student policies over trajectories sampled from the **student policy distribution**.

$$
	\mathcal{L}_{\text{OPD}}=\mathbb{E}_{s\sim d^{\pi_{s}}}[D_{\text{KL}}(\pi_{t}(\cdot|s)\|\pi_{s}(\cdot|s))]
$$

If we expand the KL divergence and remove the teacher only term (constant w.r.t. student parameters), it becomes:

$$
	\mathcal{L}_{\text{OPD}}=-\mathbb{E}_{s\sim d^{\pi_{s}}}\left[ \sum_{a}\pi _{t}(a|s)\log \pi_{s}(a|s) \right]
$$

which is simply a cross entropy imitation objective on on-policy states. The on-policy actually means the state distribution is based on the to be trained model, which is student model here. If the trajectories are sampled according to teacher distribution then it becomes off-policy and is just the KD term.

Usually OPD is combined with normal RL loss like PPO or GRPO and so on.
