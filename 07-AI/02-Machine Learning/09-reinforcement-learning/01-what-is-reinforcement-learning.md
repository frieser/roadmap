---
tags: ['ai', 'roadmap']
---

## Summary
**Reinforcement Learning (RL)** is a branch of machine learning where an autonomous **Agent** learns to make optimal decisions by interacting with an **Environment**. Unlike supervised learning, which relies on static labeled datasets, RL is based on a trial-and-error paradigm where the agent receives feedback in the form of **Rewards** or penalties, aiming to maximize cumulative reward over time.

## Detailed Explanation

Reinforcement Learning is modeled as a loop of interaction between an agent and its environment. This process is typically formalized using the **Markov Decision Process (MDP)** framework.

### **1. Agent**
The **Agent** is the learner or decision-maker. It is the entity that perceives the current situation and chooses an action. Its goal is not just to get immediate rewards, but to maximize the total reward it receives in the long run.

### **2. Environment**
The **Environment** is everything external to the agent. It is the world the agent lives in and interacts with. The environment responds to the agent's actions by providing new states and rewards.

### **3. State (S)**
A **State** is a comprehensive description of the environment at a specific point in time. It contains all the information the agent needs to make a decision. For example, in a game of chess, the state is the current configuration of all pieces on the board.

### **4. Action (A)**
An **Action** is any possible move the agent can make. The set of all possible actions available to the agent is called the **Action Space**. Actions cause the environment to transition from the current state to a new state.

### **5. Reward (R)**
The **Reward** is a numerical value sent from the environment to the agent as feedback for an action. It defines the goal of the RL problem. The agent's objective is to maximize the **Cumulative Reward** (return).
- **Positive Reward**: Encourages behavior.
- **Negative Reward (Penalty)**: Discourages behavior.

### **6. Policy (π)**
The **Policy** is the agent's strategy or "brain." It is a mapping from perceived states of the environment to actions to be taken when in those states. Policies can be:
- **Deterministic**: Always chooses the same action for a given state.
- **Stochastic**: Assigns probabilities to different actions for a given state.

### **7. Value Function (V)**
While the reward provides immediate feedback, the **Value Function** represents the long-term desirability of a state. It is an estimate of the total reward an agent can expect to accumulate in the future, starting from that state. 
- **State-Value Function (V(s))**: Value of being in state $s$.
- **Action-Value Function (Q(s, a))**: Value of taking action $a$ in state $s$ (Q-Learning).

---

## Interview Questions

**Q: What is the main difference between Supervised Learning and Reinforcement Learning?**
**A:** In Supervised Learning, the model learns from a labeled dataset provided by an external "teacher" who knows the correct answers. In Reinforcement Learning, there is no teacher; the agent learns through trial and error by interacting with the environment and receiving rewards or penalties based on its actions.

**Q: Explain the Exploration vs. Exploitation trade-off.**
**A:** This is a fundamental challenge in RL. **Exploitation** means choosing the best-known action to maximize immediate reward. **Exploration** means trying new, unknown actions to see if they lead to even better rewards in the future. An agent must balance both to ensure it doesn't get stuck in sub-optimal "local maxima."

**Q: What is a Markov Decision Process (MDP)?**
**A:** An MDP is a mathematical framework used to describe the RL problem. It consists of a set of States, Actions, Transition Probabilities (the likelihood of moving from one state to another given an action), and Rewards. It assumes the "Markov Property," meaning the future state depends only on the current state and action, not on the sequence of events that preceded it.

**Q: What is the "Reward Hypothesis"?**
**A:** The Reward Hypothesis states that all goals and purposes can be well-thought-of as the maximization of the expected value of the cumulative sum of a received scalar signal (called reward).

**Q: What is the difference between Model-based and Model-free RL?**
**A:** **Model-based RL** involves the agent trying to learn or build a model of how the environment works (the transition dynamics) to plan its actions. **Model-free RL** skips this step and learns directly from experience (e.g., Q-Learning or Policy Gradients) without explicitly modeling the environment's physics or rules.
