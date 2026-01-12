---
tags: ['ai', 'roadmap']
---

## Summary
Activation functions are mathematical equations attached to each neuron in a neural network, determining whether it should be "activated" (fired) or not based on the input. They introduce **non-linearity** into the network, allowing it to learn complex patterns and approximate any continuous function. Without non-linear activation functions, a multi-layer neural network would behave like a single-layer linear regression model.

## Detailed Explanation

### 1. Sigmoid Function
The Sigmoid function maps any real-valued number into a range between **0 and 1**. It is historically significant but less common in hidden layers today.
- **Formula**: $\sigma(x) = \frac{1}{1 + e^{-x}}$
- **Pros**: Clear physical interpretation (probability); smooth gradient.
- **Cons**: **Vanishing Gradient Problem** (gradients become near-zero for large positive/negative inputs); Output is not zero-centered.

```python
import numpy as np

def sigmoid(x):
    return 1 / (1 + np.exp(-x))

# Example
x = np.array([-2, 0, 2])
print(f"Sigmoid: {sigmoid(x)}") # Output: [0.119, 0.5, 0.880]
```

### 2. Tanh (Hyperbolic Tangent)
Tanh is similar to Sigmoid but maps inputs to the range **-1 to 1**.
- **Formula**: $\tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}}$
- **Pros**: **Zero-centered** (mean output is near 0), which makes optimization easier for the next layer.
- **Cons**: Still suffers from the vanishing gradient problem at extreme values.

```python
def tanh(x):
    return np.tanh(x)

print(f"Tanh: {tanh(x)}") # Output: [-0.964, 0.0, 0.964]
```

### 3. ReLU (Rectified Linear Unit)
ReLU is the default choice for most deep learning hidden layers. It outputs the input if it is positive, and zero otherwise.
- **Formula**: $f(x) = \max(0, x)$
- **Pros**: Computationally efficient; reduces the likelihood of vanishing gradients (gradient is 1 for $x > 0$).
- **Cons**: **Dying ReLU problem** (neurons can become inactive and only output 0 if they get stuck in the negative range).

```python
def relu(x):
    return np.maximum(0, x)

print(f"ReLU: {relu(x)}") # Output: [0, 0, 2]
```

### 4. Leaky ReLU
A variant of ReLU that attempts to solve the "Dying ReLU" problem by allowing a small, non-zero gradient when the input is negative.
- **Formula**: $f(x) = \max(\alpha x, x)$, typically $\alpha = 0.01$.
- **Pros**: Prevents dead neurons by ensuring a small gradient even for negative values.

```python
def leaky_relu(x, alpha=0.01):
    return np.where(x > 0, x, alpha * x)

print(f"Leaky ReLU: {leaky_relu(x)}") # Output: [-0.02, 0.0, 2.0]
```

### 5. Softmax
Typically used in the **output layer** of a multi-class classification network. It turns a vector of numbers into a probability distribution that sums to 1.
- **Formula**: $\sigma(\mathbf{z})_i = \frac{e^{z_i}}{\sum_{j=1}^K e^{z_j}}$
- **Usage**: Multi-class classification (e.g., classifying an image as a dog, cat, or bird).

```python
def softmax(x):
    e_x = np.exp(x - np.max(x)) # Subtract max for numerical stability
    return e_x / e_x.sum(axis=0)

logits = np.array([2.0, 1.0, 0.1])
print(f"Softmax: {softmax(logits)}") # Output: [0.659, 0.242, 0.098]
```

## Interview Questions

**Q1: Why do we need non-linear activation functions?**
**A:** Without non-linearity, the composition of multiple layers would still be a linear function (matrix multiplication). Stacking 100 linear layers would be equivalent to a single linear layer, making the network unable to learn complex non-linear relationships like those found in images or natural language.

**Q2: What is the Vanishing Gradient problem?**
**A:** It occurs when the gradients of the loss function approach zero as they are backpropagated through the network. Functions like Sigmoid and Tanh have very small derivatives at extreme values. When these small numbers are multiplied during chain rule, the update to the weights in early layers becomes negligible, effectively stopping the learning process.

**Q3: When would you use Softmax instead of Sigmoid?**
**A:** Use **Sigmoid** for binary classification (two classes, one output neuron representing probability). Use **Softmax** for multi-class classification (N classes, N output neurons representing a probability distribution across all classes).

**Q4: How does ReLU handle the vanishing gradient problem differently than Sigmoid?**
**A:** For all positive inputs, the derivative of ReLU is exactly 1. This means the gradient does not "shrink" as it passes through multiple ReLU layers (as long as neurons are active), allowing deep networks to train much faster.

**Q5: What is the "Dying ReLU" problem?**
**A:** It happens when a neuron gets pushed into the negative range where its gradient is zero. If the learning rate is too high or the weights are initialized poorly, a large portion of the network can "die," meaning those neurons never fire and never update their weights again. Leaky ReLU or ELU are common fixes.
