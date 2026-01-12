---
tags: ['ai', 'roadmap']
---

# Calculus: Derivatives and Partial Derivatives

## Summary
Derivatives and partial derivatives are the fundamental tools for optimization in Machine Learning. They quantify how a function's output changes in response to small changes in its inputs. In ML, we use them to calculate gradients, which tell us how to adjust model parameters (weights and biases) to minimize a loss function during training, a process known as Gradient Descent.

## Detailed Explanation

### 1. The Derivative (Single Variable)
The derivative of a function $f(x)$ measures the instantaneous rate of change of $f$ with respect to $x$. Geometrically, it is the slope of the line tangent to the curve at a given point.

**Formal Definition:**
$f'(x) = \lim_{h \to 0} \frac{f(x+h) - f(x)}{h}$

**Common Rules:**
- **Power Rule:** $\frac{d}{dx}x^n = nx^{n-1}$
- **Sum Rule:** $(f + g)' = f' + g'$
- **Product Rule:** $(fg)' = f'g + fg'$

### 2. Partial Derivatives (Multivariable)
In Machine Learning, functions (like Loss functions) usually depend on thousands or millions of variables (weights). A partial derivative measures the rate of change with respect to **one** variable while holding all others constant.

**Notation:** $\frac{\partial f}{\partial x}$ or $f_x$

If $f(x, y) = x^2 + y^3$, then:
- $\frac{\partial f}{\partial x} = 2x$
- $\frac{\partial f}{\partial y} = 3y^2$

### 3. Symbolic Differentiation with SymPy
SymPy allows for exact algebraic manipulation of derivatives, which is useful for theoretical analysis.

```python
import sympy as sp

# Define symbols
x, y = sp.symbols('x y')

# Define a function: f(x, y) = x^2 + 2xy + y^2
f = x**2 + 2*x*y + y**2

# Compute partial derivatives
df_dx = sp.diff(f, x)
df_dy = sp.diff(f, y)

print(f"Partial w.r.t x: {df_dx}") # 2*x + 2*y
print(f"Partial w.r.t y: {df_dy}") # 2*x + 2*y
```

### 4. Automatic Differentiation with PyTorch (Autograd)
In Deep Learning, we use numerical values and "Automatic Differentiation". PyTorch's `autograd` engine tracks all operations on tensors to compute gradients efficiently.

```python
import torch

# Create a tensor and tell PyTorch to track its gradient
x = torch.tensor(2.0, requires_grad=True)
y = torch.tensor(3.0, requires_grad=True)

# Define a function (loss): z = x^2 + y^2
z = x**2 + y**2

# Compute gradients via backpropagation
z.backward()

# Access the partial derivatives
print(f"dz/dx at x=2: {x.grad}") # 2*x = 4.0
print(f"dz/dy at y=3: {y.grad}") # 2*y = 6.0
```

### 5. Foundation of Gradient Descent
The vector containing all partial derivatives is called the **Gradient** ($\nabla f$). It points in the direction of the steepest ascent. To minimize a function (like training a model), we move in the opposite direction:
$w = w - \eta \nabla L(w)$
where $\eta$ is the learning rate and $L$ is the loss function.

## Interview Questions

**Q1: What is the geometric interpretation of a derivative?**
**A:** The derivative at a point represents the slope of the tangent line to the function's graph at that point. It indicates the rate of change and the direction in which the function is moving.

**Q2: Explain the difference between a derivative and a partial derivative.**
**A:** A derivative is used for functions of a single variable. A partial derivative is used for functions of multiple variables and measures how the function changes as one specific variable changes while all others are held constant.

**Q3: How are partial derivatives used in training neural networks?**
**A:** They are the core of the backpropagation algorithm. We compute the partial derivative of the loss function with respect to each weight and bias in the network to determine how to update them to reduce the error.

**Q4: What is the gradient vector?**
**A:** The gradient $\nabla f$ is a vector of all partial derivatives of a multivariable function. It indicates the direction of the steepest increase of the function.

**Q5: Why is Automatic Differentiation preferred over Symbolic Differentiation in Deep Learning?**
**A:** Symbolic differentiation can lead to "expression swell" (extremely long formulas) for deep neural networks. Automatic differentiation is computationally efficient, as it decomposes complex functions into simple operations and applies the chain rule numerically, allowing it to handle millions of parameters.
