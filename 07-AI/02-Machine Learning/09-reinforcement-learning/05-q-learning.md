---
tags: ['ai', 'roadmap']
---

## Summary
Q-Learning is a fundamental, model-free **Reinforcement Learning (RL)** algorithm used to learn the value of an action in a particular state. In its table-based form, it maintains a **Q-Table** that stores the expected future rewards for every state-action pair. By iteratively updating these values using the **Bellman Equation**, the agent eventually discovers the optimal policy without requiring a model of the environment's dynamics.

## Detailed Explanation

### The Core Concept: The Q-Table
In table-based Q-Learning, we represent the agent's knowledge using a 2D array (or dictionary) where:
- **Rows** represent states ($s$).
- **Columns** represent actions ($a$).
- **Cells** store the **Q-value** $Q(s, a)$, representing the "quality" or expected total reward of taking action $a$ in state $s$.

### The Bellman Equation (Update Rule)
The Q-values are updated as the agent interacts with the environment:
$$Q(s, a) \leftarrow (1 - \alpha) \cdot Q(s, a) + \alpha \cdot \left[ R + \gamma \cdot \max_{a'} Q(s', a') \right]$$

Where:
- $\alpha$ (**Learning Rate**): How much new information overrides old information (0 to 1).
- $R$: The immediate reward received after taking action $a$.
- $\gamma$ (**Discount Factor**): The importance of future rewards (0 to 1). A higher value makes the agent "far-sighted".
- $\max_{a'} Q(s', a')$: The maximum predicted reward for the next state $s'$.

### Exploration vs. Exploitation
- **Exploration**: Trying unknown actions to discover potentially better strategies.
- **Exploitation**: Using the current best-known action to maximize short-term rewards.

To balance these, we use the **Epsilon-Greedy ($\epsilon$-greedy)** strategy:
1. Generate a random number $p$ between 0 and 1.
2. If $p < \epsilon$: **Explore** (pick a random action).
3. Else: **Exploit** (pick the action with the highest Q-value for the current state).
4. Usually, $\epsilon$ decays over time so the agent explores more at the start and exploits more as it learns.

### Python Implementation Example
Using a simple grid-world context:

```python
import numpy as np
import random

# Hyperparameters
alpha = 0.1    # Learning rate
gamma = 0.9    # Discount factor
epsilon = 0.1  # Exploration rate
episodes = 1000

# Environment setup (Example: 5 states, 2 actions)
num_states = 5
num_actions = 2
q_table = np.zeros((num_states, num_actions))

def get_action(state, epsilon):
    if random.uniform(0, 1) < epsilon:
        return random.randint(0, num_actions - 1) # Explore
    else:
        return np.argmax(q_table[state]) # Exploit

# Training Loop
for _ in range(episodes):
    state = 0 # Starting state
    done = False
    
    while not done:
        action = get_action(state, epsilon)
        
        # Simulate environment (Dummy logic)
        next_state = min(state + 1, num_states - 1)
        reward = 1 if next_state == num_states - 1 else 0
        done = (next_state == num_states - 1)
        
        # Update Q-Table
        old_value = q_table[state, action]
        next_max = np.max(q_table[next_state])
        
        # Bellman Equation
        q_table[state, action] = (1 - alpha) * old_value + alpha * (reward + gamma * next_max)
        
        state = next_state

print("Learned Q-Table:")
print(q_table)
```

## Interview Questions

**Q: What is the main limitation of Table-based Q-Learning?**
**A:** The "Curse of Dimensionality". As the number of states and actions increases, the Q-Table grows exponentially, making it computationally impossible to store or explore (e.g., Chess or StarCraft). This is why Deep Q-Networks (DQN) use neural networks to approximate Q-values instead of a table.

**Q: Why do we need a Discount Factor ($\gamma$)?**
**A:** The discount factor determines how much the agent cares about future rewards compared to immediate ones. If $\gamma=0$, the agent is "myopic" and only considers immediate rewards. If $\gamma \approx 1$, it strives for long-term high rewards. It also ensures mathematical convergence in infinite-horizon tasks.

**Q: What happens if the Learning Rate ($\alpha$) is set to 1?**
**A:** If $\alpha=1$, the update rule becomes $Q(s, a) = R + \gamma \max Q(s', a')$. The agent completely ignores previously learned values and only considers the most recent experience, which usually leads to unstable learning and failure to converge in stochastic environments.

**Q: Explain the difference between Off-policy and On-policy learning (and which one is Q-Learning).**
**A:** Q-Learning is **Off-policy** because it learns the value of the optimal policy independently of the agent's actions (it uses $\max Q$ for the update, even if the agent took a random exploratory action). In contrast, **SARSA** is **On-policy** because it updates Q-values based on the actual action the agent took following its current policy.
