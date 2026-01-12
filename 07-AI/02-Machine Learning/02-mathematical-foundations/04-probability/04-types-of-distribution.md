---
tags: ['ai', 'roadmap']
---

# Types of Probability Distributions

## Summary
Probability distributions are mathematical functions that describe the likelihood of obtaining the possible values that a random variable can take. In machine learning, understanding these distributions is crucial for data modeling, hypothesis testing, and understanding the behavior of algorithms. They are broadly categorized into **Discrete** (outcomes are countable) and **Continuous** (outcomes can take any value in a range).

## Detailed Explanation

### 1. Bernoulli Distribution (Discrete)
The Bernoulli distribution represents a single trial with two possible outcomes: success (1) with probability $p$ and failure (0) with probability $q = 1-p$.

*   **Usage**: Modeling binary events (e.g., coin toss, user clicks an ad).
*   **Mean**: $p$
*   **Variance**: $p(1-p)$

```python
from scipy.stats import bernoulli

# Probability of success
p = 0.6
# Calculate probability of success (1)
prob_success = bernoulli.pmf(1, p)
print(f"Probability of success: {prob_success}")
```

### 2. Binomial Distribution (Discrete)
The Binomial distribution describes the number of successes in $n$ independent Bernoulli trials, each with the same probability of success $p$.

*   **Formula**: $P(X=k) = \binom{n}{k} p^k (1-p)^{n-k}$
*   **Usage**: Number of heads in 10 coin tosses.

```python
from scipy.stats import binom

n, p = 10, 0.5
# Probability of exactly 5 successes in 10 trials
prob_5 = binom.pmf(5, n, p)
print(f"Probability of 5 successes: {prob_5:.4f}")
```

### 3. Poisson Distribution (Discrete)
The Poisson distribution models the number of times an event occurs in a fixed interval of time or space, given the average rate of occurrence ($\lambda$).

*   **Formula**: $P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}$
*   **Usage**: Number of emails received per hour, number of website visitors per minute.

```python
from scipy.stats import poisson

lam = 3  # average events per interval
# Probability of observing exactly 5 events
prob_5 = poisson.pmf(5, lam)
print(f"Probability of 5 events (λ=3): {prob_5:.4f}")
```

### 4. Uniform Distribution (Discrete & Continuous)
In a Uniform distribution, all outcomes are equally likely within a certain range $[a, b]$.

*   **Continuous PDF**: $f(x) = \frac{1}{b-a}$
*   **Usage**: Random number generators, initialization of neural network weights.

```python
from scipy.stats import uniform

# Continuous uniform between 0 and 10
start, width = 0, 10
# Probability density at x=5
density = uniform.pdf(5, start, width)
print(f"Probability density at 5: {density}")
```

### 5. Normal / Gaussian Distribution (Continuous)
The Normal distribution is characterized by its bell-shaped curve, defined by mean ($\mu$) and standard deviation ($\sigma$). It is the most important distribution in statistics due to the **Central Limit Theorem**.

*   **Formula**: $f(x) = \frac{1}{\sigma \sqrt{2\pi}} e^{-\frac{1}{2}(\frac{x-\mu}{\sigma})^2}$
*   **Usage**: Heights, IQ scores, errors in measurements, most natural phenomena.

```python
from scipy.stats import norm

mu, sigma = 0, 1
# Probability of value being less than 1.96 (standard normal)
prob_lt_1_96 = norm.cdf(1.96, mu, sigma)
print(f"P(X < 1.96): {prob_lt_1_96:.4f}") # approx 0.975
```

### 6. Exponential Distribution (Continuous)
The Exponential distribution models the time between events in a Poisson point process (events occurring continuously and independently at a constant average rate).

*   **Formula**: $f(x) = \lambda e^{-\lambda x}$
*   **Usage**: Time until a radioactive particle decays, time between arrivals at a service desk.
*   **Property**: **Memoryless** - the probability of an event occurring in the next interval is independent of how much time has already passed.

```python
from scipy.stats import expon

# Lambda = 0.5 (scale = 1/lambda = 2)
scale = 2
# Probability of waiting less than 3 units of time
prob_lt_3 = expon.cdf(3, scale=scale)
print(f"Probability wait < 3: {prob_lt_3:.4f}")
```

## Interview Questions

**Q: What is the difference between Bernoulli and Binomial distributions?**
**A:** A Bernoulli distribution models a single trial with two outcomes (success/failure). A Binomial distribution models the number of successes in multiple ($n$) independent Bernoulli trials.

**Q: When would you use a Poisson distribution instead of a Binomial distribution?**
**A:** Use Poisson when the number of trials ($n$) is very large and the probability of success ($p$) is very small, such that $np$ is constant (represented by $\lambda$). It is ideal for counting rare events over a continuous interval.

**Q: What is the Central Limit Theorem and why is it important for the Normal distribution?**
**A:** The Central Limit Theorem states that the sum (or average) of a large number of independent and identically distributed random variables will tend toward a Normal distribution, regardless of the original distribution. This justifies why many real-world variables are normally distributed and allows us to use Gaussian-based statistical tests.

**Q: What does the "memoryless" property of the Exponential distribution mean?**
**A:** It means that the probability of an event occurring in the future does not depend on how much time has already elapsed. Formally: $P(X > s+t | X > s) = P(X > t)$.

**Q: How do you identify if a dataset follows a Normal distribution?**
**A:** You can use visual methods like **Histograms** or **Q-Q plots**, or statistical tests like the **Shapiro-Wilk test** or **Kolmogorov-Smirnov test**.
