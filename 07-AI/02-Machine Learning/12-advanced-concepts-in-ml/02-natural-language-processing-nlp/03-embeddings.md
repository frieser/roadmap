---
tags: ['ai', 'roadmap']
---

## Summary
**Word Embeddings** are dense vector representations of words in a high-dimensional continuous space. Unlike traditional sparse representations (like one-hot encoding), embeddings capture semantic and syntactic relationships between words, allowing models to understand that "king" is to "queen" as "man" is to "woman" through vector arithmetic.

## Detailed Explanation

### 1. Word2Vec (Predictive Model)
Developed by Google, Word2Vec uses a shallow two-layer neural network to learn word associations from a large corpus of text.
- **CBOW (Continuous Bag of Words):** Predicts a target word based on the surrounding context words. It is faster to train and has slightly better accuracy for frequent words.
- **Skip-gram:** Predicts the surrounding context words given a single target word. It works well with small amounts of training data and represents rare words or phrases well.

### 2. GloVe (Global Vectors for Word Representation)
Developed by Stanford, GloVe is a "count-based" model. It builds a global co-occurrence matrix that tabs how often words appear together in a dataset. 
- It combines the benefits of global matrix factorization (like LSA) with the local context window methods (like Word2Vec).
- The training objective is to learn word vectors such that their dot product equals the log of the words' probability of co-occurrence.

### 3. FastText (Subword Embeddings)
Developed by Facebook (Meta), FastText is an extension of Word2Vec.
- **Character n-grams:** Instead of treating each word as an atomic unit, FastText treats each word as a bag of character n-grams (e.g., "apple" with n=3: `<ap`, `app`, `ppl`, `ple`, `le>`).
- **Out-of-Vocabulary (OOV):** This allows FastText to generate embeddings for words not seen during training by aggregating the vectors of its character n-grams.
- It is particularly effective for morphologically rich languages (like Finnish or Turkish).

### 4. Contextual Embeddings
Static embeddings (Word2Vec, GloVe, FastText) assign the same vector to a word regardless of its meaning in a specific sentence (polysemy). Contextual embeddings solve this:
- **ELMo (Embeddings from Language Models):** Uses a deep bi-directional LSTM to generate embeddings based on the entire sentence.
- **BERT (Bidirectional Encoder Representations from Transformers):** Uses the Transformer architecture to look at both the left and right context of a word simultaneously.
- **Result:** The word "bank" will have a different embedding in "river bank" than in "bank deposit".

## Interview Questions

### 1. What is the "King - Man + Woman = Queen" example?
This demonstrates **linear analogical reasoning** in embedding spaces. It shows that word embeddings capture semantic relationships as geometric translations. The vector difference between "King" and "Man" encodes the concept of "royalty" or "gender," which when added to "Woman" results in a vector closest to "Queen."

### 2. Why are word embeddings preferred over One-Hot Encoding?
One-hot encoding creates extremely high-dimensional and sparse vectors that treat all words as equidistant (orthogonal), failing to capture any semantic similarity. Word embeddings are dense (low-dimensional, typically 100-300), computationally efficient, and cluster semantically related words together.

### 3. How does FastText handle the Out-of-Vocabulary (OOV) problem?
By breaking words down into character n-grams, FastText can construct a vector for an unseen word by summing the vectors of the n-grams it contains. This is impossible for Word2Vec or GloVe, which would return an error or a generic "unknown" token vector.

### 4. What is the main limitation of static embeddings like GloVe?
The main limitation is **Polysemy**. Since each word has exactly one fixed vector, the model cannot distinguish between different meanings of the same word based on context (e.g., "fine" as in "doing fine" vs "a parking fine").

### 5. What is the difference between a Count-based model and a Predictive model?
Count-based models (like GloVe) learn by factorizing a global co-occurrence matrix, focusing on global statistics. Predictive models (like Word2Vec) learn by trying to predict a word from its neighbors, focusing on local context windows.
