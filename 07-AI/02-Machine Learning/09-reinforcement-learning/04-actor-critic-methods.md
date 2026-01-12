---
tags: ['ai', 'roadmap']
---

## Summary
**Actor-Critic methods** are a hybrid class of Reinforcement Learning algorithms that combine the benefits of both **Policy-based** (Actor) and **Value-based** (Critic) methods. The **Actor** updates the policy distribution in the direction suggested by the **Critic**, which estimates the value function to reduce variance in the policy gradient. **A2C (Advantage Actor-Critic)** and **A3C (Asynchronous Advantage Actor-Critic)** are the most prominent implementations, leveraging the **Advantage function** to improve training stability and efficiency.

## Detailed Explanation

### 1. The Core Architecture
In Actor-Critic methods, the agent is split into two components:
- **The Actor**: Responsible for selecting actions. It learns a policy $\pi_\theta(a|s)$ using policy gradient methods.
- **The Critic**: Responsible for evaluating actions. It learns a value function $V_\phi(s)$ (or $Q_\phi(s, a)$) to provide feedback to the actor.

### 2. The Advantage Function
To reduce the high variance associated with vanilla Policy Gradient (REINFORCE), Actor-Critic methods use the **Advantage Function** $A(s, a)$:
$$A(s, a) = Q(s, a) - V(s)$$
The Advantage tells us how much better an action is compared to the average action in that state. In practice, we often use the **Temporal Difference (TD) Error** as an unbiased estimate of the Advantage:
$$\delta = r + \gamma V(s') - V(s)$$

### 3. A2C vs. A3C
- **A3C (Asynchronous Advantage Actor-Critic)**:
    - Multiple independent workers interact with their own copies of the environment.
    - Workers compute gradients locally and update a **global network** asynchronously.
    - Diversity of experience from different workers breaks the correlation between samples.
- **A2C (Advantage Actor-Critic)**:
    - The synchronous version of A3C.
    - A coordinator waits for all workers to finish their segment of experience, averages the gradients, and then updates the global network.
    - Often more efficient on GPUs and easier to implement than A3C.

### 4. Implementation Example (PyTorch)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim

class ActorCritic(nn.Module):
    def __init__(self, state_dim, action_dim, hidden_size=256):
        super(ActorCritic, self).__init__()
        self.affine = nn.Linear(state_dim, hidden_size)
        
        # Actor head: output probability distribution over actions
        self.action_head = nn.Linear(hidden_size, action_dim)
        
        # Critic head: output scalar state-value V(s)
        self.value_head = nn.Linear(hidden_size, 1)

    def forward(self, x):
        x = F.relu(self.affine(x))
        
        action_prob = F.softmax(self.action_head(x), dim=-1)
        state_values = self.value_head(x)
        
        return action_prob, state_values

def compute_loss(action_probs, state_values, returns):
    # returns: discounted sum of rewards
    # log_probs: log policy for actions taken
    
    # Calculate Advantage
    advantages = returns - state_values.detach()
    
    # Actor Loss: -log_pi * Advantage (negative for gradient ascent)
    policy_loss = -(torch.log(action_probs) * advantages).mean()
    
    # Critic Loss: MSE between predicted V(s) and actual returns
    value_loss = F.mse_loss(state_values, returns)
    
    # Total Loss (can include entropy bonus for exploration)
    return policy_loss + 0.5 * value_loss
```

## Interview Questions

1. **What is the main advantage of Actor-Critic methods over vanilla Policy Gradient?**
   - **Answer**: Actor-Critic methods significantly reduce variance by using a Critic (Value Function) as a baseline. Instead of relying on full trajectory returns (which are noisy), the Actor uses the Advantage function (often estimated via TD error) provided by the Critic to update the policy.

2. **Explain the difference between A2C and A3C.**
   - **Answer**: A3C is **asynchronous**; multiple workers update a global model at different times, which can lead to "stale" gradients. A2C is **synchronous**; it waits for all workers to finish their batch before performing a single synchronized update. A2C is often preferred for GPU utilization.

3. **Why do we include an Entropy term in the Actor-Critic loss function?**
   - **Answer**: The entropy term ($H(\pi(s))$) encourages exploration by preventing the policy from prematurely converging to a deterministic distribution (where one action has probability 1.0). High entropy means the distribution is more spread out.

4. **What does the Advantage Function $A(s, a) = Q(s, a) - V(s)$ represent intuitively?**
   - **Answer**: It represents the "relative" benefit of taking a specific action $a$ in state $s$ compared to the average performance of the current policy in that state. If $A(s, a) > 0$, the action is better than average; if $A(s, a) < 0$, it is worse.
