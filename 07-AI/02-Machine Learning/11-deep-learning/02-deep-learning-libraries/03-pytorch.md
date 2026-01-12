---
tags: ['ai', 'roadmap']
---

# PyTorch

## Summary
**PyTorch** is an open-source machine learning framework based on the Torch library, primarily developed by Meta's AI Research lab (FAIR). It is celebrated for its flexibility, ease of use, and "Pythonic" design. Unlike frameworks that use static graphs, PyTorch utilizes a **Dynamic Computational Graph** (Define-by-Run), which allows for more intuitive debugging and the construction of complex, varying architectures. It has become the leading framework in both academic research and industry for deep learning applications.

## Detailed Explanation

### 1. Tensors: The Foundation
At its core, PyTorch provides the `Tensor` object, which is very similar to NumPy's `ndarray`. The key differences are:
- **GPU Acceleration**: Tensors can be easily moved to GPUs for massive speedups in computation.
- **Automatic Differentiation**: Tensors track their operations to enable gradient calculation.

```python
import torch

# Create a tensor from data
data = [[1, 2], [3, 4]]
x_data = torch.tensor(data)

# Move to GPU if available
if torch.cuda.is_available():
    x_data = x_data.to('cuda')

print(f"Tensor Shape: {x_data.shape}")
print(f"Device: {x_data.device}")
```

### 2. Dynamic Computational Graph (Define-by-Run)
The standout feature of PyTorch is its **Dynamic Computational Graph**. 
- **Static Graphs (Define-and-Run)**: The graph is defined once and then executed repeatedly (common in early TensorFlow).
- **Dynamic Graphs (Define-by-Run)**: The graph is built on-the-fly during the forward pass. This means every execution can have a different graph structure.

**Benefits**:
- **Easier Debugging**: You can use standard Python debuggers (like `pdb`) and print statements.
- **Dynamic Architectures**: Easily handle variable-length inputs (e.g., in RNNs) or conditional logic within the model.

### 3. Autograd: Automatic Differentiation
The `torch.autograd` library powers PyTorch’s ability to calculate gradients, which is essential for backpropagation.
- When a tensor's `requires_grad` attribute is `True`, PyTorch tracks all operations on it.
- Calling `.backward()` on a scalar (usually the loss) computes the gradients of all tensors in the graph.

```python
x = torch.ones(2, 2, requires_grad=True)
y = x + 2
z = y * y * 3
out = z.mean()

out.backward() # Computes gradients
print(x.grad)  # d(out)/dx
```

### 4. Neural Networks with `torch.nn`
PyTorch uses the `nn.Module` class as the base for all neural network components. You define the layers in `__init__` and the data flow in `forward`.

```python
import torch.nn as nn
import torch.nn.functional as F

class SimpleNet(nn.Module):
    def __init__(self):
        super(SimpleNet, self).__init__()
        self.fc1 = nn.Linear(784, 128)
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x):
        x = F.relu(self.fc1(x))
        x = self.fc2(x)
        return x

model = SimpleNet()
```

### 5. Training Loop Workflow
Training in PyTorch is explicit, giving the developer full control:
1. **Forward Pass**: Compute predictions.
2. **Loss Calculation**: Compare predictions to labels.
3. **Zero Gradients**: `optimizer.zero_grad()` to clear previous gradients.
4. **Backward Pass**: `loss.backward()` to compute new gradients.
5. **Optimizer Step**: `optimizer.step()` to update weights.

## Interview Questions

### 1. What is the difference between a Dynamic and a Static Computational Graph?
**Answer**: In a static graph (Define-and-Run), the architecture is defined and compiled before training, which allows for aggressive optimization but makes debugging harder. In a dynamic graph (Define-by-Run), the graph is built as operations are executed. This allows for standard Python control flow (loops, conditionals) inside the forward pass and easier debugging.

### 2. What is the difference between `torch.Tensor` and `torch.tensor`?
**Answer**: `torch.Tensor` is the main tensor class (alias for `torch.FloatTensor` by default). `torch.tensor` is a factory function that infers the data type from the input data and copies the data into the new tensor. It is generally recommended to use `torch.tensor` for creating tensors from data.

### 3. Why is `optimizer.zero_grad()` necessary in a PyTorch training loop?
**Answer**: PyTorch accumulates gradients by default. If you don't call `zero_grad()`, the gradients from the current batch will be added to the gradients from the previous batch, leading to incorrect weight updates. Accumulation is sometimes useful (e.g., for effective larger batch sizes), but in standard loops, it must be cleared.

### 4. How do you prevent PyTorch from tracking gradients for evaluation?
**Answer**: You use the `with torch.no_grad():` context manager. This disables gradient tracking, which reduces memory consumption and speeds up computations during inference or model evaluation.

### 5. What are the main components of a PyTorch `Dataset` class?
**Answer**: To create a custom dataset, you inherit from `torch.utils.data.Dataset` and must implement three methods:
- `__init__`: For initialization (loading file paths, etc.).
- `__len__`: Returns the total number of items in the dataset.
- `__getitem__`: Returns a single sample (and its label) at a given index.
