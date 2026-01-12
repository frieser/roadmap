---
tags: ['ai', 'roadmap']
---

## Summary
The **Attention Mechanism** revolutionized Natural Language Processing by addressing the "bottleneck" problem in traditional **Seq2Seq** (Sequence-to-Sequence) models. Instead of compressing an entire input sentence into a single fixed-length vector, attention allows the decoder to "look back" at different parts of the input sequence at each step of the output generation. This results in significantly better performance, especially for long sequences, and laid the foundation for the Transformer architecture.

## Detailed Explanation

### 1. The Seq2Seq Bottleneck
Traditional Seq2Seq models consist of an **Encoder** and a **Decoder**. The encoder processes the input and produces a final hidden state (the "context vector"), which is supposed to summarize the entire input. The decoder then generates the output based *only* on this single vector.
- **Problem**: For long sentences, a single fixed-length vector cannot capture all relevant information. This is known as the **bottleneck problem**.

### 2. Bahdanau Attention (Additive Attention)
Introduced by Bahdanau et al. (2014), this was the first widely successful attention mechanism.
- **Concept**: The context vector is computed as a weighted sum of all encoder hidden states ($h_i$).
- **Alignment Score**: It uses a small feedforward neural network to calculate how well the previous decoder state $s_{t-1}$ matches each encoder state $h_i$.
- **Formula**: $score(s_{t-1}, h_i) = v_a^T \tanh(W_a [s_{t-1}; h_i])$
- **Key Characteristics**:
    - **Additive**: The states are concatenated and passed through a linear layer.
    - **Context Before State**: The context vector is calculated *before* determining the current decoder state $s_t$.

### 3. Luong Attention (Multiplicative Attention)
Proposed by Luong et al. (2015), this model refined and simplified the attention process.
- **Alignment Scores**: It introduced more efficient ways to calculate attention:
    - **Dot-product**: $s_t^T h_i$
    - **General**: $s_t^T W_a h_i$
    - **Concat**: $v_a^T \tanh(W_a [s_t; h_i])$
- **Key Characteristics**:
    - **Multiplicative**: Uses matrix multiplication, which is faster and more space-efficient.
    - **Context After State**: The current decoder hidden state $s_t$ is computed first, then used to calculate the attention context.
- **Global vs. Local Attention**:
    - **Global Attention**: Attends to all source positions (similar to Bahdanau).
    - **Local Attention**: Attends only to a small window of source positions, reducing computational cost.

### 4. Comparison Table

| Feature | Bahdanau Attention | Luong Attention |
| :--- | :--- | :--- |
| **Score Function** | Additive (Feed-forward) | Multiplicative (Dot-product/General) |
| **Decoder State** | Uses $s_{t-1}$ | Uses $s_t$ |
| **Alignment** | Global only | Global or Local |
| **Complexity** | More complex (parameters in MLP) | Simpler and faster |

## Interview Questions

### 1. What is the "bottleneck problem" in Seq2Seq models, and how does Attention solve it?
In traditional Seq2Seq, the encoder must compress all input information into a single fixed-length vector. For long sequences, this leads to information loss. Attention solves this by allowing the decoder to access all encoder hidden states selectively, focusing on relevant parts of the input for each generated word.

### 2. How do Bahdanau and Luong attention differ in their use of decoder hidden states?
Bahdanau attention uses the **previous** decoder hidden state ($s_{t-1}$) to calculate the alignment scores and context vector. In contrast, Luong attention first calculates the **current** decoder hidden state ($s_t$) and then uses it to derive the attention weights.

### 3. Explain the difference between Global and Local attention in the Luong model.
**Global attention** considers all hidden states of the encoder when calculating the context vector, which is thorough but computationally expensive for long sequences. **Local attention** identifies a specific "aligned" position in the input and only considers a small window of words around that position, making it more efficient.

### 4. Why is multiplicative attention (Luong) generally preferred over additive attention (Bahdanau)?
Multiplicative attention uses highly optimized matrix multiplication operations, making it faster and more memory-efficient in practice. While both perform similarly in terms of accuracy, Luong's simplified architecture is often easier to implement and scale.

### 5. What role did these attention models play in the development of the Transformer?
These models introduced the concept of "Soft Attention," where the model learns weights for different inputs. The Transformer took this further by replacing recurrent layers (RNNs/LSTMs) entirely with "Self-Attention," allowing for massive parallelism and better handling of long-range dependencies.
