Before talking about the methods, we must understand why we keep working on such topic. The answer is, we want better **performance**, faster **speed** and **stability** or **reliability**.

Post training comes from the work of large language model's alignment and safety work. It is usually applied after supervised fine-tuning.

# PPO

PPO (Proximal Policy Optimization) is greatly inspired by TRPO (Trust Region Policy Optimization) which optimize the policy by maximize the following equation with constraint:

$$
\begin{align}
\max_{\theta} \;&
\hat{\mathbb{E}}_{t}\!\left[
\frac{\pi_{\theta}(a_t \mid s_t)}
     {\pi_{\theta_{\text{old}}}(a_t \mid s_t)}
\hat{A}_t
\right] \\
\text{subject to } \;&
\hat{\mathbb{E}}_{t}\!\left[
\mathrm{KL}\!\left(
\pi_{\theta_{\text{old}}}(\cdot \mid s_t),
\pi_{\theta}(\cdot \mid s_t)
\right)\right] \le \delta
\end{align}
$$

we can transform the object with something like Lagrangian dual into a unconstrained problem.

However solving for the divergence constraint is pretty hard and takes more computation time, so why do we need this constraint? It is designed to ensure the update of policy is not too big. We can imagine if there is no constraint, and for a timestamp the advantage $A$ is very big, the policy will try it's best to maximize that action, especially when the old probability is pretty low. And then the training will be very unstable, so we need the divergence. And that is why PPO is very clever.

Since our main purpose is to **_reduce shift in objective_**, PPO propose they will **_just clip objective directly_**. By doing so, they don't need to solve for the divergence which means faster speed, easier to implement and can still achieve a pretty good performance, stability and reliability.

# DPO

The full name of DPO is Direct Preference Optimization, which is proposed for Reinforcement Learning Human Feedback (RLHF). It is usually trained with human-annotated preference data $(q,o_{+},o_{-})\sim\mathcal{D}_{\text{DPO}}$. The loss function is:

$$
\mathcal{L}_{\text{DPO}} = -\mathbb{E}_{(q, o_+, o_-) \sim \mathcal{D}_{\text{DPO}}} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(o_+|q)}{\pi_{\text{ref}}(o_+|q)} - \beta \log \frac{\pi_\theta(o_-|q)}{\pi_{\text{ref}}(o_-|q)} \right) \right]
$$

Since it is trained with human-annotated data, it is also considered as a supervised training process. And that's why it has faster speed but worse performance on more complex tasks.

# GRPO

Full name: Group Relative Policy Optimization. It is used in [[DeepSeek-R1]] and achieved great success. Instead of maintaining a value network (which is used to calculate $A$, the advantage) like PPO, GRPO generates a group of $G$ trajectories for each prompt and normalizes the corresponding rewards within each group to compute the advantages:

$$
\mathcal{J}_{\text{GRPO}}(\theta) = \underset{\substack{q \sim \mathcal{Q} \\ o_i \sim \pi_{\theta_{\text{old}}}}}{\mathbb{E}} \, \frac{1}{G} \sum_{i=1}^G \frac{1}{|o_i|} \sum_{t=1}^{|o_i|} \min\left[ \frac{\pi_\theta(o_{i,t} | o_{i,<t}, q)}{\pi_{\theta_{\text{old}}}(o_{i,t} | o_{i,<t}, q)} A_{i,t}, \, \text{clip}\left( \frac{\pi_\theta(o_{i,t} | o_{i,<t}, q)}{\pi_{\theta_{\text{old}}}(o_{i,t} | o_{i,<t}, q)}, 1-\epsilon, 1+\epsilon \right) A_{i,t} \right]
$$

and the advantage is calculated with:

$$
A_{i,t}= \frac{r_{i}-\text{mean}(\mathbf{r})}{\text{std}(\mathbf{r})+\epsilon}
$$

So we reduce the cost of a value model, only use reward model to calculate the advantage, which improves speed and maintain pretty good performance.

## Problems of GRPO

### Breaking baseline rule

GRPO calculates advantage by subtracting mean of rewards and dividing it with standard deviation, but the baseline rule of reinforcement learning says that, what we can do is to subtract any _state-dependent only_ term from out rewards. So this breaks such rule, and the key problem is dividing by standard deviation. With low deviation we will have much greater reward, so model will learn more from it. And usually very easy (all correct) and very hard (all wrong) questions can achieve low deviation, such design of advantage leads to **bias towards super easy and hard questions.**

### Length bias

GRPO takes output length $|o_{i}|$ into consideration, which means when we have negative rewards, the model will try to make the answer as long as possible, in contrast when rewards are positive, the model will try to make short answers. That is probably why we see really long chain of thoughts in DeepSeek's GRPO trained model, this is not proven but a quite interesting point of view.

# GSPO

## Motivation

The motivation of GSPO is the basic disadvantage of off-policy learning. All the policy optimization methods we have talked about are off-policy methods, which means we are getting experiences based on older version of the policy and then apply the total update rather than small cycles of inference and update. But this leads to an deviation in objective learning. So we developed various methods to restrict the gradient estimation, like clipping. But the problem still exists, especially with the background of training super big models. From the research, they find out that it is because GRPO only applies the importance weight $\frac{\pi_{\theta}(y_{i,t}\mid x,y_{i,<t})}{\pi_{\theta_{\text{old}}}(y_{i,t}\mid x,y_{i,<t})}$ at each token position $t$. Since this weight is based on a single sample $y_{i,t}$ from each next token distribution, it fails to perform the intended distribution-correction role. The failure of token-level importance weight points to a **core principle**: the unit of optimization objective should match the unit of of reward. Since the reward is granted to the entire sequence, we should explore performing optimization directly at the _sequence level_.

## Algorithm

The sequence level importance weight will be: $\frac{\pi_{\theta}(y\mid x)}{\pi_{\theta_{\text{old}}}(y\mid x)}$. Then they modify the GRPO $r(\theta)$ to $s(\theta)$ based on sequence likelihood:

$$
s_i(\theta) = \left( \frac{\pi_\theta(y_i|x)}{\pi_{\theta_{\text{old}}}(y_i|x)} \right)^{\frac{1}{|y_i|}} = \exp\left( \frac{1}{|y_i|} \sum_{t=1}^{|y_i|} \log \frac{\pi_\theta(y_{i,t}|x, y_{i,<t})}{\pi_{\theta_{\text{old}}}(y_{i,t}|x, y_{i,<t})} \right).
$$

Through experiments, GSPO shows better efficiency, accuracy than GRPO and enable model to have continuous performance improvement.
