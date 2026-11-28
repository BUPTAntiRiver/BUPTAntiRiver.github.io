Actor Critic model is designed to solve continuous action space problems.
In discrete problems, we can pick the best action with highest Q value by using a simple $\arg\max$ method. However in continuous space, we cannot do that, so we train a model to find the policy that can maximize $E_{\mathbf{a}\sim \pi(\mathbf{a}|\mathbf{s})}Q(\mathbf{s},\mathbf{a})$.
The model we talked above is called *Actor*, and the Q function model is *Critic*.
# Details
## Target Critic
Target critic is designed to **prevent our goal from changing**. This problem emerges because our critic is changed during training, then the goal is also changed, to keep our model in track, we use *target critic* to memorize the goal and let it be updated less often than critic.
## Advantage
Advantage represents how much the action is **better** than the mean action.
$$
\begin{align}
&A(s,a)=Q(s,a)-V(s) \\
&V(s)=E_{a\sim \pi(a|s)}[Q(s,a)]
\end{align}
$$
So we want the action chose by our **actor** to maximize **advantage**.