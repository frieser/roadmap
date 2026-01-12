---
tags: ['ai', 'roadmap']
---

## Summary
Policy Gradient methods are a class of Reinforcement Learning (RL) algorithms that optimize the agent's policy directly, rather than relying on a value function to derive actions. The **REINFORCE** algorithm (also known as Monte Carlo Policy Gradient) is the foundational method in this family. It uses the "log-derivative trick" to estimate the gradient of the expected return and updates policy parameters via gradient ascent. Unlike Q-Learning, which is value-based and often off-policy, Policy Gradient methods are typically on-policy and can naturally handle continuous action spaces and learn stochastic policies.

## Detailed Explanation

### 1. Optimizing the Policy Directly
In value-based methods like Q-Learning, we learn a value function $Q(s, a)$ and pick actions greedily. In Policy Gradient methods, we define a parameterized policy $\pi_\theta(a|s)$ (e.g., a neural network) that outputs a probability distribution over actions. Our goal is to maximize the expected total reward:
$$J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} [R(\tau)]$$
where $\tau$ is a trajectory $(s_0, a_0, r_1, s_1, \dots)$ and $R(\tau)$ is the cumulative return.

### 2. Stochastic Policies
Policy Gradient methods require the policy to be **stochastic**. This allows the agent to explore the environment and provides a differentiable way to calculate how changes in $\theta$ affect the probability of actions. 
- **Discrete Actions**: Usually modeled with a Categorical distribution (Softmax output).
- **Continuous Actions**: Usually modeled with a Gaussian distribution (Mean and Standard Deviation outputs).

### 3. The Log-Derivative Trick
Calculating $\nabla_\theta J(\theta)$ is difficult because the expectation depends on the policy $\pi_\theta$, which affects the distribution of trajectories. The log-derivative trick solves this:
$$\nabla_\theta \pi_\theta = \pi_\theta \frac{\nabla_\theta \pi_\theta}{\pi_\theta} = \pi_\theta \nabla_\theta \log \pi_\theta$$
Applying this to the expectation yields the **Policy Gradient Theorem**:
$$\nabla_\theta J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[ \sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) G_t \right]$$
where $G_t$ is the return from time $t$. This allows us to estimate the gradient by simply sampling trajectories and calculating the log-probability of the actions taken, weighted by the resulting reward.

### 4. REINFORCE Implementation (PyTorch)
The following example demonstrates a simple REINFORCE update loop for a discrete action space.

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.distributions import Categorical

class PolicyNetwork(nn.Module):
    def __init__(self, state_dim, action_dim):
        super(PolicyNetwork, self).__init__()
        self.fc = nn.Sequential(
            nn.Linear(state_dim, 128),
            nn.ReLU(),
            nn.Linear(128, action_dim),
            nn.Softmax(dim=-1)
        )

    def forward(self, x):
        return self.fc(x)

def select_action(policy, state):
    state = torch.from_numpy(state).float().unsqueeze(0)
    probs = policy(state)
    m = Categorical(probs)
    action = m.sample()
    return action.item(), m.log_prob(action)

def update_policy(optimizer, saved_log_probs, rewards, gamma=0.99):
    R = 0
    policy_loss = []
    returns = []
    
    # Calculate returns (G_t)
    for r in rewards[::-1]:
        R = r + gamma * R
        returns.insert(0, R)
    
    returns = torch.tensor(returns)
    # Standardizing returns helps reduce variance
    returns = (returns - returns.mean()) / (returns.std() + 1e-9)
    
    for log_prob, Gt in zip(saved_log_probs, returns):
        # We use negative for gradient ascent in a descent optimizer
        policy_loss.append(-log_prob * Gt)
    
    optimizer.zero_grad()
    policy_loss = torch.cat(policy_loss).sum()
    policy_loss.backward()
    optimizer.step()

# Usage in a training loop:
# log_probs, rewards = [], []
# action, lp = select_action(policy, state)
# log_probs.append(lp)
# rewards.append(reward)
# ... end of episode ...
# update_policy(optimizer, log_probs, rewards)
```

## Interview Questions

**Q: What is the main disadvantage of REINFORCE?**
**A:** High variance. Because the gradient is estimated using the total return of a single trajectory, small changes in the environment or stochasticity in the policy can lead to very different returns, making the gradient updates noisy and training unstable. This is usually mitigated by using a **baseline** (e.g., subtracting a value function $V(s)$ from the return).

**Q: Why do Policy Gradient methods prefer $\nabla \log \pi$ over $\nabla \pi$?**
**A:** The log-derivative trick $\frac{\nabla \pi}{\pi} = \nabla \log \pi$ allows us to convert the gradient of an expectation into an expectation of a gradient. Practically, it "normalizes" the gradient: instead of just increasing the probability of good actions, it scales the update by the inverse of the current probability, ensuring that rare but high-reward actions get a significant boost.

**Q: How does REINFORCE handle continuous action spaces?**
**A:** Instead of outputting a probability for each discrete action, the neural network outputs the parameters of a continuous distribution, typically the mean $\mu$ and standard deviation $\sigma$ of a Gaussian. Actions are sampled from $\mathcal{N}(\mu, \sigma)$, and the gradient is calculated with respect to $\mu$ and $\sigma$ using the same log-probability principle.

**Q: Is REINFORCE an on-policy or off-policy algorithm?**
**A:** It is strictly **on-policy**. The gradient estimate requires that the trajectories used for the update were sampled using the *current* policy parameters $\theta$. If we used trajectories from an old policy, the gradient estimate would be biased unless we used Importance Sampling.
