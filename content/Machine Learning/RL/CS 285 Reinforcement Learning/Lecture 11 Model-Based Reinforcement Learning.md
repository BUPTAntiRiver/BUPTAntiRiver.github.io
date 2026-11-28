In this lecture we will introduce basic model-based $\text{RL}$. Which means we learn a model, and use model for control.
# Why learn the model?
Because in the methods we have learned so far, they all requires the knowledge about what next state we will be after taken given action under current state. This is a must.
We can do practical experiments to collect data, but that could be very **expensive** if we need to do so every time.
With a dynamics model $f(\mathbf{s},\mathbf{a})$ we can plan through it easily.

## *model-based reinforcement learning version 0.5*:
1. run base policy $\pi_{0}(\mathbf{a}_{t}|\mathbf{s}_{t})$ (e.g., random policy) to collect $\mathcal{D}=\{(\mathbf{s},\mathbf{a},\mathbf{s}')_{i}\}$
2. learn dynamics model $f(\mathbf{s}, \mathbf{a})$ to minimize $\sum_{i}||f(\mathbf{s}_{i},\mathbf{a}_{i})-\mathbf{s}_{i}'||^2$
3. plan through $f(\mathbf{s},\mathbf{a})$ to choose actions

However, $p_{\pi_{f}}(\mathbf{s}_{t})\neq p_{\pi_{0}}(\mathbf{s}_{t})$, this is called the Distribution mismatch problem.
This problem appears because training distribution is collected based on a simple policy, it only visited limited amount of states, so the policy we get can only performs well on these states, in other words, it has bad generalization performance.
For example, we are at an ascending hill with a cliff. The simple policy runs randomly only on the ascending part, so the dynamics model we get only tells information about ascending hill, then the trained policy we get only learns to move forward to get higher rewards. But when it means a new situation, around the edge of cliff, the policy haven't seen that before, it doesn't know what to do, so it choose to move forward because he always do so, then it falls and gets much lower rewards.

To solve this problem, we need to collect data from $p_{\pi_{f}}(\mathbf{s}_{t})$.
## *model-based reinforcement learning version 1.0*:
1. run base policy $\pi_{0}(\mathbf{a}_{t}|\mathbf{s}_{t})$ (e.g., random policy) to collect $\mathcal{D}=\{(\mathbf{s},\mathbf{a},\mathbf{s}')_{i}\}$
2. learn dynamics model $f(\mathbf{s}, \mathbf{a})$ to minimize $\sum_{i}||f(\mathbf{s}_{i},\mathbf{a}_{i})-\mathbf{s}_{i}'||^2$
3. plan through $f(\mathbf{s},\mathbf{a})$ to choose actions
4. execute those actions and add the resulting data $\{(\mathbf{s},\mathbf{a},\mathbf{s}')_{j}\}$ to $\cal D$
repeat step 2 to 4

We can do better by **re-planning when model errors**.
## *model-based reinforcement learning version 1.0*:
1. run base policy $\pi_{0}(\mathbf{a}_{t}|\mathbf{s}_{t})$ (e.g., random policy) to collect $\mathcal{D}=\{(\mathbf{s},\mathbf{a},\mathbf{s}')_{i}\}$
2. learn dynamics model $f(\mathbf{s}, \mathbf{a})$ to minimize $\sum_{i}||f(\mathbf{s}_{i},\mathbf{a}_{i})-\mathbf{s}_{i}'||^2$
3. plan through $f(\mathbf{s},\mathbf{a})$ to choose actions
4. execute the first planned action, observe resulting state $\mathbf{s}'$ (Model Predictive Control)
5. append $\{(\mathbf{s},\mathbf{a},\mathbf{s}')_{j}\}$ to dataset $\cal D$
small loop: step 3 to 5, big loop (for every N steps): step 2 to 5
### How to re-plan?
The more you re-plan, the less perfect each individual plan need to be.
So we can use shorter horizons, and even random sampling can often work here!
# Uncertainty in Model-Based RL
Uncertainty can help the model to fit a wider range of functions, this is also the meaning of planning all the actions, that means, we will take actions for which we think we'll get high reward in expectation while maybe high variance but this avoids "exploiting" the model and makes the model adapt and get better.
# Uncertainty-Aware Neural Net Models
## *Model-Based reinforcement learning with latent state*:
1. run base policy $\pi_{0}(\mathbf{a}_{t}|\mathbf{o}_{t})$ (e.g., random policy) to collect data $\mathcal{D}=\{(\mathbf{o},\mathbf{a},\mathbf{o}')_{i}\}$
2. learn $p_{\phi}(\mathbf{s}_{t+1}|\mathbf{s}_{t},\mathbf{a}_{t}),p_{\phi}(r_{t}|\mathbf{s}_{t}),p(\mathbf{o}_{t}|\mathbf{s}_{t}),g_{\psi}(\mathbf{o}_{t})$
3. plan through the model to choose actions
4. execute the first planned action, observe resulting $\mathbf{o}'$ ($\text{MPC}$)
5. append $(\mathbf{o}, \mathbf{ a}, \mathbf{o}')$ to dataset $\mathcal{D}$
small loop: step 3 to 5, big loop: step 2 to 5
