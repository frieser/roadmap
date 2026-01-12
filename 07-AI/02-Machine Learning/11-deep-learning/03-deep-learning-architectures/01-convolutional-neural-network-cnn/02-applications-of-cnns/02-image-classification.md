---
tags: ['ai', 'roadmap']
---

## Summary
**Image Classification** is a fundamental Computer Vision task where a model assigns one or more labels to an input image from a predefined set of categories. Convolutional Neural Networks (CNNs) are the state-of-the-art approach for this task because they can automatically learn spatial hierarchies of features, from low-level edges to high-level complex objects, making them highly effective at recognizing patterns in visual data.

## Detailed Explanation

Image classification typically falls into two main categories based on the nature of the labels assigned to the images:

### 1. Single-label (Multi-class) Classification
In single-label classification, each image belongs to exactly one category. For example, in the **MNIST** dataset, an image represents a single digit from 0 to 9.
*   **Final Layer Activation**: **Softmax**. This function scales the raw output scores (logits) into a probability distribution where the sum of all probabilities equals 1.
*   **Loss Function**: **Categorical Crossentropy** (or Sparse Categorical Crossentropy if labels are integers).
*   **Output Interpretation**: The class with the highest probability is chosen as the prediction.

### 2. Multi-label Classification
In multi-label classification, an image can contain multiple objects or attributes simultaneously (e.g., an image containing a "sunset," "beach," and "person").
*   **Final Layer Activation**: **Sigmoid**. Each output node is treated as an independent binary classification problem (Presence vs. Absence).
*   **Loss Function**: **Binary Crossentropy** applied to each output node.
*   **Output Interpretation**: Any class with a probability above a certain threshold (e.g., 0.5) is considered present.

### The Role of the Architecture
A typical CNN for classification consists of:
1.  **Feature Extraction**: Multiple Convolutional and Pooling layers that detect features.
2.  **Flattening**: Converting the 2D feature maps into a 1D vector.
3.  **Classification (Head)**: Fully Connected (Dense) layers that map the extracted features to the final class probabilities.

### Python Example: MNIST Classification with TensorFlow/Keras
The following example demonstrates a simple CNN architecture for the MNIST digit classification task.

```python
import tensorflow as tf
from tensorflow.keras import layers, models

# 1. Load and preprocess the MNIST dataset
mnist = tf.keras.datasets.mnist
(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0  # Normalize to [0, 1]

# Add a channels dimension (MNIST is grayscale)
x_train = x_train[..., tf.newaxis].astype("float32")
x_test = x_test[..., tf.newaxis].astype("float32")

# 2. Build the CNN Model
model = models.Sequential([
    # Feature Extraction
    layers.Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1)),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    
    # Flattening
    layers.Flatten(),
    
    # Classification Head
    layers.Dense(64, activation='relu'),
    layers.Dropout(0.2), # Regularization
    layers.Dense(10, activation='softmax') # 10 classes for digits 0-9
])

# 3. Compile the model
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

# 4. Train the model
model.fit(x_train, y_train, epochs=5, batch_size=64, validation_split=0.1)

# 5. Evaluate
test_loss, test_acc = model.evaluate(x_test, y_test, verbose=2)
print(f'\nTest accuracy: {test_acc:.4f}')
```

## Interview Questions

**Q: What is the difference between Softmax and Sigmoid activation functions in the context of image classification?**
**A:** Softmax is used for **single-label (multi-class)** classification because it ensures that the sum of all output probabilities is 1, creating a competition between classes where only one is the "winner." Sigmoid is used for **multi-label** classification (or binary classification) because it treats each output independently, allowing multiple classes to have high probabilities simultaneously.

**Q: Why do we use Categorical Crossentropy for MNIST but Binary Crossentropy for a multi-label tagger?**
**A:** Categorical Crossentropy measures the performance of a classification model whose output is a probability distribution that sums to 1 (mutually exclusive classes). Binary Crossentropy measures the performance of a model where each output is an independent probability, which is required when an image can belong to multiple categories at once.

**Q: What is the purpose of the "Flatten" layer in a CNN classifier?**
**A:** The Flatten layer converts the multi-dimensional feature maps (e.g., 7x7x64) produced by the convolutional and pooling layers into a single long 1D vector. This is necessary because the subsequent Fully Connected (Dense) layers expect a 1D input to perform the final classification mapping.

**Q: How does a CNN handle images of different sizes for classification?**
**A:** CNNs typically require a fixed input size because the Fully Connected layers at the end have a fixed number of weights. To handle different sizes, images are usually **resized** or **cropped** during preprocessing. Alternatively, techniques like **Global Average Pooling (GAP)** can be used instead of Flattening to make the network size-agnostic for the feature extraction part.

**Q: What is Class Imbalance and how can you mitigate it in image classification?**
**A:** Class imbalance occurs when some classes have significantly more samples than others. This can lead to a biased model. Mitigation strategies include **oversampling** the minority class, **undersampling** the majority class, using **weighted loss functions** (giving more importance to rare classes), or using data augmentation to synthesize new samples for the minority class.
