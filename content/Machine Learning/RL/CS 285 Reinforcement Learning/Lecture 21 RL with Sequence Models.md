# Beyond MDPs
So far only states and actions are discussed and they **obey Markov property well**, but now, I am going to introduce partial observations, which you might have seen in the very early part of this course. And observations **do not obey the Markov property**.
Let me explain:
In the scene of a cheetah chasing a goat, the states might be the position, velocity and all kinds of physical properties about the 2 animals, but our observations can only be a picture of the scene. When there is some blocking which makes you can not see the animals, then previous observations can help you predict the future like how human do in real life. And that is not allowed in Markov, because you are making predictions basing on states other than current state.
**Most real world problems are like this!**
P.S. This seems like context...
## Which methods handle partial observability?
I have some candidates for you:
- Policy Gradients
- Value-based methods
- Model-based RL methods
What I mean by handle is that these methods don't depend on Markov property.
### Policy Gradients
$$
\nabla _{\theta}J(\theta)\approx \frac{1}{N}\sum^N_{i=1}\sum^T_{t=1}\nabla_{\theta}\log \pi_{\theta}(\mathbf{a}_{i,t}|\mathbf{o}_{i,t})A(\mathbf{o}_{i,t},\mathbf{a}_{i,t})
$$
This is the general form of policy gradient. The policy part is OK for partial observability, because it does not use Markov property, but $A$ advantage function depends.
#### Case 1:
$$
A(\mathbf{o}_{i,t},\mathbf{a}_{i,t})=\sum^T_{t=1}r(\mathbf{o}_{i,t},\mathbf{a}_{i,t})
$$
This is fine.
#### Case 2:
$$
A(\mathbf{o}_{i,t},\mathbf{a}_{i,t})=r_{i,t}+\gamma \hat{V}(\mathbf{o}_{i,t+1})-\hat{V}(\mathbf{o}_{i,t})
$$
This is not okay, because it does not use observations before step $t$.
#### Case 3:
$$
A(\mathbf{o}_{i,t},\mathbf{a}_{i,t})=\sum^T_{t'=t}r(\mathbf{o}_{i,t'},\mathbf{a}_{i,t'})
$$
This is also OK, quite surprising, right? Though here we didn't use observations before step $t$, but the core idea is whether we use Markov property or not. Here we only calculate rewards, and rewards have no relationship with Markov at all.
### Value Based Models
$$
Q(\mathbf{o},\mathbf{a})\leftarrow r(\mathbf{o},\mathbf{a})+\gamma\max_{\mathbf{a}'}Q(\mathbf{o}',\mathbf{a}')
$$
It doesn't work without Markov property.
### Model-Based RL methods
Applying partial observation in model-based RL is also a bad idea.
Let me give you a example:
There are 2 doors, one of them is chosen randomly to be open, the other one is locked. So the model will learn $p(\mathbf{o}'=\text{Pass}|\mathbf{o}=\text{Left},\mathbf{a}=\text{Open})=0.5$. Because it is trained under a lot of data. But, in real world, the after the unlocked door is chosen, it won't change! But the model might just *always try to open the door on the left* because the probabilities are the same. So the model might gets wrong all the time and that is the reason why **Model-Based RL methods rely on Markov property**.

## History States
We can use one state to represent history observations, which makes the state a function of history observations.
$$
\mathbf{s}_{t}=(\mathbf{o}_{1},\mathbf{o}_{2},\dots,\mathbf{o}_{t})
$$
This obeys Markov property!
# RL and language models
Language models are *typically* trained on supervised learning. But we can also train them on RL.
# Multi-step RL and language models
## What is a time step?
### Per-utterance

### Per-token