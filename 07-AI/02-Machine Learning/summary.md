---
tags: ['ai', 'roadmap', 'ml', 'summary', 'schematic']
---
## Track Overview
| 114 files | 6 levels deep | 12 domain groups |

## Mathematical Foundations
| Domain | Key Concepts |
|---|---|
| Linear Algebra (5) | Matrix ops, SVD (A=UΣV^T), eigenvalues, determinants, tensors |
| Calculus (3) | Partial derivatives → gradient; chain rule → backprop; Jacobian/Hessian |
| Probability (4) | Bayes' theorem, PDF/CDF, Gaussian/Bernoulli/Poisson distributions |
| Statistics (3) | Descriptive (mean/σ), inferential (p-value/t-test), visualization |
| Discrete Math (1) | Sets, graph theory → Bayesian nets, decision trees |

## Programming & Libraries
|---|---|
| Python (7) | Syntax, data types, loops, conditionals, exceptions, functions, OOP |
| NumPy | N-D arrays, linear algebra, broadcasting |
| Pandas | DataFrame, groupby, merge, time series |
| Matplotlib/Seaborn | 2D plots; statistical viz: heatmaps, pair plots |
## Data Pipeline
| Step | Methods |
|---|---|
| Sources (5) | SQL/NoSQL, web/APIs, mobile, IoT streams |
| Formats (5) | CSV, JSON, Parquet, Excel, HDF5/Avro |
| Cleaning | Impute missing, IQR/Z-score outliers, deduplicate, type conversion |

## ML Core
| Concept | Summary |
|---|---|
| Paradigms (3) | Supervised (labeled), Unsupervised (structure), Semi/self-supervised |
| Scikit-learn (6) | load → split → prep → select → tune → predict |
| Feature Eng (5) | Encode categoricals, scale (Standard/MinMax/Robust), select (filter/wrapper/embedded), PCA/LDA/t-SNE |

## Supervised Learning
| Algorithm | Trait |
|---|---|
| KNN | Lazy, distance-based, scale-sensitive |
| Logistic Regression | Linear boundary, sigmoid → probability, log loss |
| SVM | Max-margin, kernel trick (RBF/poly), tune C+γ |
| Decision Trees | Gini/entropy splits, prone to overfit → prune |
| Random Forest | Bagging + feature randomness, OOB error |
| GBM (XGB/LGBM/CatBoost) | Sequential residual fitting, regularization, categorical support |
| Linear Regression | OLS: min Σ(y-ŷ)²; assumes linearity, homoscedasticity |
| Polynomial/Lasso/Ridge/ElasticNet | Add x² terms; L1 zeros coefficients; L2 shrinks; combine |

## Unsupervised Learning
| Type | Algorithms |
|---|---|
| Clustering (4) | K-Means (exclusive), Fuzzy C-Means/GMM (overlapping), Agglomerative (hierarchical), GMM (probabilistic) |
| Dim Reduction (2) | PCA (linear, variance-max), Autoencoders (non-linear, compression) |

## Reinforcement Learning
| Method | Mechanism |
|---|---|
| Q-Learning | Model-free, Q-table, off-policy |
| DQN | Q-Learning + neural net for state approx |
| Policy Gradient | Direct policy optimization, REINFORCE |
| Actor-Critic | Value fn (critic) + policy fn (actor) combined |

## Model Evaluation
| Metric | Formula | Use |
|---|---|---|
| Accuracy | (TP+TN)/total | Balanced classes |
| Precision | TP/(TP+FP) | Min false positives |
| Recall | TP/(TP+FN) | Min false negatives |
| F1 | 2PR/(P+R) | Imbalanced classes |
| ROC-AUC | TPR vs FPR area | Rank quality |
| Log Loss | -[y log(p)+(1-y)log(1-p)] | Calibration |
| Confusion Matrix | TP,TN,FP,FN | Full breakdown |

| Validation | How |
|---|---|
| K-Fold CV | K splits; train K-1, test 1; average |
| LOOCV | K=N; unbiased but expensive |

## Deep Learning
| Component | Key Facts |
|---|---|
| Perceptron/MLP | Single neuron → feedforward layers |
| Forward/Backprop | Input → output; chain rule gradients backwards |
| Activation: ReLU | max(0,x) — default; avoids vanishing gradient |
| Loss: MSE / Cross-Entropy | Regression / Classification |

| Library | Trait |
|---|---|
| TensorFlow | Static graph, production serving |
| PyTorch | Dynamic graph, research, Pythonic |
| Keras | High-level API on TF |

### Deep Learning Architectures
| Architecture | Mechanism | Domain |
| CNN | Conv → pooling → stride; translation invariant | Images |
| RNN/LSTM/GRU | Sequential memory; gates (forget/input/output) | Sequences |
| Self-Attention | QK^T/√d_k → softmax → V-weighted sum | NLP/Transformers |
| Multi-Head Attention | H parallel attention heads → concatenated | BERT, GPT |
| Autoencoders | Encode → bottleneck → decode; reconstruction loss | Denoising, anomaly |
| GANs | Generator vs Discriminator minimax; mode collapse risk | Image generation |

## Advanced Concepts
| Topic | Key Points |
|---|---|
| XAI | LIME (local surrogate), SHAP (Shapley values); interpretability vs accuracy tradeoff |
| Tokenization | Subword (BPE/WordPiece/SentencePiece) solves OOV; preferred over word-level for LLMs |
| Lemmatization vs Stemming | Dictionary+POS → valid word vs heuristic chop → fast but imprecise |
| Embeddings | Word2Vec (CBOW/Skip-gram), GloVe, FastText (char n-grams), BERT contextual |
| Attention (Seq2Seq) | Bahdanau (additive, pre-state) → Luong (multiplicative, post-state) → Transformers |

## Rules
1. Scale features for KNN/SVM/LR/NN; trees are scale-invariant
2. Fit scaler on training set only; transform test — prevents leakage
3. KNN: K ≈ √n, odd K for binary; curse of dimensionality sensitive
4. Small SVM C = wide margin (bias↑); large C = tight margin (overfit)
5. Learning rate = most critical GBM hyperparameter; tune first
6. Cross-Entropy for classification, MSE for regression; never MSE+sigmoid
7. ReLU default activation; sigmoid only for binary output layer
8. Self-attention: O(n²·d); Lasso→feature select; Ridge→shrink coefficients
9. LIME/SHAP for black-box explanation; prefer glass-box (LR/DT) when feasible
10. L1 (Lasso) → feature selection (zero coefficients); L2 (Ridge) → shrink
