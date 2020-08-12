---
title: T-SNE
date: 2019-10-04
tags:
    - machine learning
categories:
    - Learn
summary: 一种降维方法，将高维数据映射到低维空间，用于可视化。
---

## Prepare

- Perplexity
  Measurement of how well a probability model predicts a sample. 
  - Definition 
    $$H(P) = -\sum_x p(x) \log_2 p(x)$$
    $$\text{Perplexity} = 2^{H(p)}$$
  - Understanding 
    - $H(p)$ be the average of bits to decode the information
    - $\text{Perplexity}$ is the total amount of all possible information
  - Usage
    In the NLP, the best language model is one that predicts an unseen test set gives the highest P
      $$ PP(w) = p (w_1w_2\dots w_N )^{-1/N}$$
      Minmizing perplexity is the same as maximizing probability
- Gaussian Distriubtion and T Distribution
  - T distribution is used when the samples is small
  - T distribution is heavy tail
    ![Student_t_pdf.svg](https://upload.wikimedia.org/wikipedia/commons/4/41/Student_t_pdf.svg)

## T-SNE method

### SNE method (Stochastic Neighbor Embedding)

1. High Dimension Space
    {{<formula>}}
    $$p_{j | i}=\frac{\exp \left(-\left\|x_{i}-x_{j}\right\|^{2} / 2 \sigma_{i}^{2}\right)}{\sum_{k \neq i} \exp \left(-\left\|x_{i}-x_{k}\right\|^{2} / 2 \sigma_{i}^{2}\right)}$$
    {{</formula>}}

2. Low dimension space

    {{<formula>}}
    $$q_{j | i}=\frac{\exp \left(-\left\|y_{i}-y_{j}\right\|^{2}\right)}{\sum_{k \neq i} \exp \left(-\left\|y_{i}-y_{k}\right\|^{2}\right)}$$
    {{</formula>}}

3. Cost function
    {{<formula>}}
    $$C=\sum_{i} K L\left(P_{i} \| Q_{i}\right)=\sum_{i} \sum_{j} p_{j | i} \log \frac{p_{j | i}}{q_{j | i}}$$
    {{</formula>}}

4. How to choose $\sigma$ for different points

    Different region have different density, so the $\sigma$ determines how many effective neighbors needs to be considered. As the $\sigma$ increase, the entropy of the distribution is increased. And the entroy has an expontential form with **perplexity**. Using the **perplexity** to determine the $\sigma$ with every point has a fixed **perplexity**.
  
- Problem
  - The cost function is not symmetric, it focus on retaining the local structure of the data
  - Dealing with outlier
  - Crowding problem : The area of the two-dimensional map that is available to accommodate distant datapoints will not be large enough compared with the area available to accommodate nearby datapoints
  
### T-SNE advantages

1. Symmetric SNE

  - Pairwise similarities in the low dimensional map $q_{ij}$
    {{<formula>}}
    $$q_{i j}=\frac{\exp \left(-\left\|y_{i}-y_{j}\right\|^{2}\right)}{\sum_{k \neq l} \exp \left(-\left\|y_{k}-y_{l}\right\|^{2}\right)}$$
    {{</formula>}}
  - High dimensional $p_{ij}$
    {{<formula>}}
    $$P_{ij} = \frac {p_{j|i} + p_{i|j}}{2n}$$
    {{</formula>}}

2. Crowding problem
  <img src="/img/tsne_01.png"/>
  The gradient could be negative, so the low dimensional space could be expanded, and the heavy tail T-Distribution is preferred.

3. Easy to compute the gradient

    {{<formula>}}
    $$\frac{\delta C}{\delta y_{i}}=4 \sum_{j}\left(p_{i j}-q_{i j}\right)\left(y_{i}-y_{j}\right)\left(1+\left\|y_{i}-y_{j}\right\|^{2}\right)^{-1}$$
    {{</formula>}}

## Algorithm

- Data : data Set $\mathcal{X} = \{x_1, x_2, \dots, x_n\}$
- cost function parameters: perplexity $Perp$
- optimzation paraemters: number of iterations $T$, learning rate $\eta$, momentum $\alpha(t)$
- Output: low-dimensional data representation $\mathcal{Y}^{T} = \{y_1, y_2, \dots, y_n\}$
- Steps
  1. compute pairwise affinities $p_{j|i}$ with perplexity $Perp$
  2. Set $p\_{ij} = (p\_{j|i} + p\_{i|j}) / 2n$, sample initial solution $\mathcal{Y}^{(0)} = {y\_1, y\_2, \dots, y\_n}$ from $\mathcal{N}(0, 10^{-4}I)$
  3. for $t = 1$ to $T$ do
      - compute low-dimensional affinities $q_ij$
      - compute gradient $\partial C / \partial \mathcal{Y}$
      - set
        $$\mathcal{Y}^{(t)}=\mathcal{Y}^{(t-1)}+\eta \frac{\delta C}{\delta \gamma}+\alpha(t)\left(\mathcal{Y}^{(t-1)}-\mathcal{Y}^{(t-2)}\right)$$
 
## Code with python

```python
import numpy as np
import pylab


def Hbeta(D=np.array([]), beta=1.0):
    """
        Compute the perplexity and the P-row for a specific value of the
        precision of a Gaussian distribution.
    """

    # Compute P-row and corresponding perplexity
    P = np.exp(-D.copy() * beta)
    sumP = sum(P)
    H = np.log(sumP) + beta * np.sum(D * P) / sumP
    P = P / sumP
    return H, P


def x2p(X=np.array([]), tol=1e-5, perplexity=30.0):
    """
        Performs a binary search to get P-values in such a way that each
        conditional Gaussian has the same perplexity.
    """

    # Initialize some variables
    print("Computing pairwise distances...")
    (n, d) = X.shape
    sum_X = np.sum(np.square(X), 1)
    D = np.add(np.add(-2 * np.dot(X, X.T), sum_X).T, sum_X)
    P = np.zeros((n, n))
    beta = np.ones((n, 1))
    logU = np.log(perplexity)

    # Loop over all datapoints
    for i in range(n):

        # Print progress
        if i % 500 == 0:
            print("Computing P-values for point %d of %d..." % (i, n))

        # Compute the Gaussian kernel and entropy for the current precision
        betamin = -np.inf
        betamax = np.inf
        Di = D[i, np.concatenate((np.r_[0:i], np.r_[i+1:n]))]
        (H, thisP) = Hbeta(Di, beta[i])

        # Evaluate whether the perplexity is within tolerance
        Hdiff = H - logU
        tries = 0
        while np.abs(Hdiff) > tol and tries < 50:

            # If not, increase or decrease precision
            if Hdiff > 0:
                betamin = beta[i].copy()
                if betamax == np.inf or betamax == -np.inf:
                    beta[i] = beta[i] * 2.
                else:
                    beta[i] = (beta[i] + betamax) / 2.
            else:
                betamax = beta[i].copy()
                if betamin == np.inf or betamin == -np.inf:
                    beta[i] = beta[i] / 2.
                else:
                    beta[i] = (beta[i] + betamin) / 2.

            # Recompute the values
            (H, thisP) = Hbeta(Di, beta[i])
            Hdiff = H - logU
            tries += 1

        # Set the final row of P
        P[i, np.concatenate((np.r_[0:i], np.r_[i+1:n]))] = thisP

    # Return final P-matrix
    print("Mean value of sigma: %f" % np.mean(np.sqrt(1 / beta)))
    return P


def pca(X=np.array([]), no_dims=50):
    """
        Runs PCA on the NxD array X in order to reduce its dimensionality to
        no_dims dimensions.
    """

    print("Preprocessing the data using PCA...")
    (n, d) = X.shape
    X = X - np.tile(np.mean(X, 0), (n, 1))
    (l, M) = np.linalg.eig(np.dot(X.T, X))
    Y = np.dot(X, M[:, 0:no_dims])
    return Y


def tsne(X=np.array([]), no_dims=2, initial_dims=50, perplexity=30.0):
    """
        Runs t-SNE on the dataset in the NxD array X to reduce its
        dimensionality to no_dims dimensions. The syntaxis of the function is
        `Y = tsne.tsne(X, no_dims, perplexity), where X is an NxD NumPy array.
    """

    # Check inputs
    if isinstance(no_dims, float):
        print("Error: array X should have type float.")
        return -1
    if round(no_dims) != no_dims:
        print("Error: number of dimensions should be an integer.")
        return -1

    # Initialize variables
    X = pca(X, initial_dims).real
    (n, d) = X.shape
    max_iter = 1000
    initial_momentum = 0.5
    final_momentum = 0.8
    eta = 500
    min_gain = 0.01
    Y = np.random.randn(n, no_dims)
    dY = np.zeros((n, no_dims))
    iY = np.zeros((n, no_dims))
    gains = np.ones((n, no_dims))

    # Compute P-values
    P = x2p(X, 1e-5, perplexity)
    P = P + np.transpose(P)
    P = P / np.sum(P)
    P = P * 4.									# early exaggeration
    P = np.maximum(P, 1e-12)

    # Run iterations
    for iter in range(max_iter):

        # Compute pairwise affinities
        sum_Y = np.sum(np.square(Y), 1)
        num = -2. * np.dot(Y, Y.T)
        num = 1. / (1. + np.add(np.add(num, sum_Y).T, sum_Y))
        num[range(n), range(n)] = 0.
        Q = num / np.sum(num)
        Q = np.maximum(Q, 1e-12)

        # Compute gradient
        PQ = P - Q
        for i in range(n):
            dY[i, :] = np.sum(np.tile(PQ[:, i] * num[:, i], (no_dims, 1)).T * (Y[i, :] - Y), 0)

        # Perform the update
        if iter < 20:
            momentum = initial_momentum
        else:
            momentum = final_momentum
        gains = (gains + 0.2) * ((dY > 0.) != (iY > 0.)) + \
                (gains * 0.8) * ((dY > 0.) == (iY > 0.))
        gains[gains < min_gain] = min_gain
        iY = momentum * iY - eta * (gains * dY)
        Y = Y + iY
        Y = Y - np.tile(np.mean(Y, 0), (n, 1))

        # Compute current value of cost function
        if (iter + 1) % 10 == 0:
            C = np.sum(P * np.log(P / Q))
            print("Iteration %d: error is %f" % (iter + 1, C))

        # Stop lying about P-values
        if iter == 100:
            P = P / 4.

    # Return solution
    return Y


if __name__ == "__main__":
    print("Run Y = tsne.tsne(X, no_dims, perplexity) to perform t-SNE on your dataset.")
    print("Running example on 2,500 MNIST digits...")
    X = np.loadtxt("mnist2500_X.txt")
    print(X.shape)
    labels = np.loadtxt("mnist2500_labels.txt")
    Y = tsne(X, 2, 50, 20.0)
    pylab.scatter(Y[:, 0], Y[:, 1], 20, labels)
    pylab.show()
```

## Outcome for mnist dataset

<img src="/img/tsne_02.png">

## Reference

1. [Perplexity Intuition (and its derivation) - Towards Data Science](https://towardsdatascience.com/perplexity-intuition-and-derivation-105dd481c8f3)
2. [GitHub - lvdmaaten/bhtsne: Barnes-Hut t-SNE](https://github.com/lvdmaaten/bhtsne)
3. [t-SNE – Laurens van der Maaten](http://lvdmaaten.github.io/tsne/)
