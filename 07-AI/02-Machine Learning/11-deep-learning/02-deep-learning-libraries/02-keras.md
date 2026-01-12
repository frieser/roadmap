---
tags: ['ai', 'roadmap']
---

## Summary
Keras is a high-level deep learning API designed for human beings, emphasizing modularity, speed, and ease of use. Originally built as a wrapper for multiple backends and later integrated into TensorFlow as \`tf.keras\`, Keras 3 has returned to its multi-backend roots. It now supports **JAX**, **PyTorch**, and **TensorFlow**, allowing developers to write model code once and run it on their framework of choice.

## Detailed Explanation
Keras provides three main design patterns to build neural networks, ranging from high-level simplicity to low-level flexibility.

### 1. Sequential API
The **Sequential API** is the most basic way to build a model. It represents a linear stack of layers where each layer has exactly one input tensor and one output tensor. It is ideal for 90% of common deep learning tasks like basic classification or regression.

\`\`\`python
import keras
from keras import layers

# Creating a model by passing a list of layers
model = keras.Sequential([
    layers.Input(shape=(784,)),
    layers.Dense(64, activation="relu", name="layer1"),
    layers.Dense(10, activation="softmax", name="layer2"),
])

# Or by adding layers incrementally
model = keras.Sequential()
model.add(layers.Dense(64, activation="relu"))
model.add(layers.Dense(10, activation="softmax"))
\`\`\`

### 2. Functional API
The **Functional API** is more powerful and flexible. It can handle models with non-linear topologies, shared layers, and multiple inputs/outputs. It treats layers as "functions" that transform tensors into other tensors.

\`\`\`python
import keras
from keras import layers

# Define the input
inputs = keras.Input(shape=(784,))

# Define layers as functions
x = layers.Dense(64, activation="relu")(inputs)
x = layers.Dense(64, activation="relu")(x)
outputs = layers.Dense(10, activation="softmax")(x)

# Create the model instance
model = keras.Model(inputs=inputs, outputs=outputs, name="mnist_model")
\`\`\`

### 3. Model Subclassing
**Model Subclassing** offers the highest level of control. You define your layers in the \`__init__\` method and the forward pass in the \`call\` method. This is the preferred method for researchers and complex custom architectures where you need to implement logic that doesn't fit into a static graph.

\`\`\`python
import keras
from keras import layers

class MyModel(keras.Model):
    def __init__(self, num_classes=10):
        super().__init__()
        # Define internal layers
        self.dense1 = layers.Dense(64, activation="relu")
        self.dense2 = layers.Dense(num_classes, activation="softmax")

    def call(self, inputs):
        # Define the forward pass logic
        x = self.dense1(inputs)
        return self.dense2(x)

model = MyModel(num_classes=10)
\`\`\`

### Keras 3: Multi-Backend Support
Keras 3 allows you to choose the backend engine at runtime. This is achieved via the \`keras.ops\` library, which provides a NumPy-like API that works across frameworks.

\`\`\`python
import os
# Set the backend BEFORE importing keras
os.environ["KERAS_BACKEND"] = "jax" 

import keras
import keras.ops as ops

# This operation will run using JAX
val = ops.ones((3, 3))
\`\`\`

## Interview Questions

**Q: What is the difference between the Sequential and Functional API?**
**A:** The Sequential API is for simple, linear stacks of layers. The Functional API allows for complex model topologies, such as multiple inputs, multiple outputs, and shared layers (e.g., residual connections or Siamese networks).

**Q: How do you implement a custom layer in Keras?**
**A:** You subclass the \`keras.layers.Layer\` class and implement the \`__init__\` (to define state/weights), \`build\` (to define weights based on input shape), and \`call\` (to define the transformation) methods.

**Q: What is the role of \`model.compile()\`?**
**A:** It configures the training process by specifying the optimizer, the loss function, and the metrics to monitor. In Keras 3, this also triggers the JIT compilation if using JAX or TensorFlow.

**Q: Why would you use Model Subclassing over the Functional API?**
**A:** Use Model Subclassing when you need full control over the model's behavior, such as implementing a custom training loop, complex conditional logic in the forward pass, or when you want a coding style similar to PyTorch or object-oriented programming.

**Q: How does Keras 3 handle cross-framework compatibility?**
**A:** Keras 3 uses a backend-agnostic abstraction layer. When you use Keras layers or operations, they are translated into the native operations of the selected backend (JAX, PyTorch, or TensorFlow) without adding significant overhead.
