---
tags: ['ai', 'roadmap']
---

# Deep Q-Networks (DQN)

## Summary
**Deep Q-Network (DQN)** is a reinforcement learning algorithm that combines Q-Learning with deep neural networks to handle high-dimensional state spaces (like raw pixels). Introduced by DeepMind in 2013/2015, it solved the instability issues of using function approximators in RL through two key innovations: **Experience Replay** and **Target Networks**. It famously learned to play Atari games at a superhuman level using only visual input.

## Detailed Explanation

### 1. From Q-Learning to DQN
In standard Q-Learning, we maintain a table of $Q(s, a)$ values. However, for environments with large or continuous state spaces, this table becomes infeasible. DQN approximates the Q-function using a neural network:
$$Q(s, a; \theta) \approx Q^*(s, a)$$
where $\theta$ represents the weights of the network.

### 2. Experience Replay
Neural networks often struggle with the "non-stationary" nature of RL data (consecutive states are highly correlated). DQN stores transitions $(s, a, r, s')$ in a **Replay Buffer**.
- **Benefits**: Breaks correlations between consecutive samples, improves data efficiency (reusing transitions), and prevents the network from "forgetting" old experiences.
- **Process**: During training, a mini-batch is sampled uniformly from the buffer to perform a gradient descent step.

### 3. Target Network
To compute the TD-target $y = r + \gamma \max_{a'} Q(s', a'; \theta)$, we use the same network we are updating. This creates a "moving target" problem, leading to oscillations or divergence.
- **Solution**: Use a separate **Target Network** with weights $\theta^-$ to calculate the target $y$:
$$y = r + \gamma \max_{a'} Q(s', a'; \theta^-)$$
- **Update**: $\theta^-$ is kept frozen and only updated to match the online network $\theta$ every $C$ steps (hard update) or via Polyak averaging (soft update).

### 4. Implementation Example (PyTorch)

```python
import torch
import torch.nn as nn
import torch.optim as optim
import random
import numpy as np
from collections import deque

class DQN(nn.Module):
    def __init__(self, state_dim, action_dim):
        super(DQN, self).__init__()
        self.fc = nn.Sequential(
            nn.Linear(state_dim, 64),
            nn.ReLU(),
            nn.Linear(64, 64),
            nn.ReLU(),
            nn.Linear(64, action_dim)
        )

    def forward(self, x):
        return self.fc(x)

class DQNAgent:
    def __init__(self, state_dim, action_dim, lr=1e-3, gamma=0.99, epsilon=1.0):
        self.q_net = DQN(state_dim, action_dim)
        self.target_net = DQN(state_dim, action_dim)
        self.target_net.load_state_dict(self.q_net.state_dict())
        self.optimizer = optim.Adam(self.q_net.parameters(), lr=lr)
        self.memory = deque(maxlen=10000)
        self.gamma = gamma
        self.epsilon = epsilon
        self.action_dim = action_dim

    def select_action(self, state):
        if random.random() < self.epsilon:
            return random.randint(0, self.action_dim - 1)
        state_t = torch.FloatTensor(state).unsqueeze(0)
        with torch.no_grad():
            return self.q_net(state_t).argmax().item()

    def train_step(self, batch_size=32):
        if len(self.memory) < batch_size: return
        
        batch = random.sample(self.memory, batch_size)
        s, a, r, s_next, done = zip(*batch)

        s = torch.FloatTensor(np.array(s))
        a = torch.LongTensor(a).unsqueeze(1)
        r = torch.FloatTensor(r).unsqueeze(1)
        s_next = torch.FloatTensor(np.array(s_next))
        done = torch.FloatTensor(done).unsqueeze(1)

        # Current Q values
        current_q = self.q_net(s).gather(1, a)
        
        # Target Q values (using target network)
        with torch.no_grad():
            max_next_q = self.target_net(s_next).max(1)[0].unsqueeze(1)
            target_q = r + (self.gamma * max_next_q * (1 - done))

        loss = nn.MSELoss()(current_q, target_q)
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()
```

## Interview Questions

**Q: Why do we use a separate Target Network in DQN?**
**A:** Using the same network to calculate both the current value and the target value leads to instability. It's like a dog chasing its own tail—the objective moves every time the network updates. A frozen target network provides a stable objective for the online network to converge towards.

**Q: What is the "Deadly Triad" in Reinforcement Learning, and how does DQN relate to it?**
**A:** The Deadly Triad consists of **Function Approximation**, **Bootstrapping**, and **Off-policy learning**. When all three are present, RL training can easily diverge. DQN uses all three but stabilizes them using Experience Replay and Target Networks.

**Q: How does the Epsilon-Greedy strategy change over time during training?**
**A:** We typically start with a high $\epsilon$ (e.g., 1.0) to encourage **exploration** of the environment. As the agent learns, we decay $\epsilon$ towards a small value (e.g., 0.01) to prioritize **exploitation** of the learned optimal actions.

**Q: What are the main limitations of the vanilla DQN?**
**A:** Vanilla DQN often suffers from **overestimation bias** (consistently overvaluing actions). This is addressed by **Double DQN**, which uses the online network to select the action and the target network to evaluate it. Other improvements include **Dueling DQN** (separating state value and action advantage) and **Prioritized Experience Replay**.
