---
title: "Generate and Visualize Ellipsoids (Ellipses)"
date: 2024-11-17 18:32:00
tags: [math, misc]
mathjax: true
---

Generate high-dimensional ellipsoids with bounded condition number, and visualize 2D ellipses using Matplotlib.

<!-- more -->

## Generating an Ellipsoid

A ellipsoid in $d$-dimensional Euclidean space can be characterized as:
$$
x \in \mathbb{R}^d :\ (x - c)^T Q (x - c) \le 1,
$$
where $Q\in \mathbb{R}^{d\times d}$ is a symmetric positive-definite matrix, and $c\in \mathbb{R}^d$ is the center of the ellipsoid. The vector $c$ is easy to construct, so the main problem becomes: how do we generate a $d\times d$ symmetric p.d. matrix? 

Let $A\in \mathbb{R}^{d\times d}$ be a random matrix, then it is almost surely full-rank. Consequently, for any nonzero $x$, we have $Ax = \sum_i x_i a_i \ne 0$, and $x^T A^T A x = ||Ax||^2 > 0$. Therefore, $A^T A$ is a p.d. matrix. To generate a p.d. matrix, we can generate the random matrix $A$ and compute $A^T A$. The code (using Eigen) is as follows:

```cpp
typedef Eigen::MatrixXd Matrix;

std::random_device rd;
std::mt19937 engine(rd());

// Generate a real symmetric p.d. matrix
Matrix GeneratePDMatrix(int d) {
	std::uniform_real_distribution<double> distribution(-1.0, 1.0);
	Matrix A = Matrix::NullaryExpr(d, d, [&](){return distribution(engine);});
	return A.transpose() * A;
}
```

However, in practice this method encountered some numerical problems in high-dimensions (when $d\ge 1000$). My suspicion is that we lost control on the eigenvalues of $A^T A$. They can be either very small or very large, and the condition number is unbounded. Below is a figure shows the distribution of the eigenvalues for 100 p.d. matrices generated using the approach.

![](/images/ellipsoids-generation-visualization/density.png)

Another approach is to generate an orthonormal matrix $P$, and a positive diagonal matrix $\Lambda$. Then the matrix $P\Lambda P^T$ is a symmetric p.d. matrix. The rows of $P$ are the eigenvectors, and the entries of $\Lambda$ are the eigenvalues. In such a way we can control the range of the eigenvalues of $Q$. 

Let $A\in \mathbb{R}^{d\times d}$ be a randomly generated full-rank matrix. We can make it orthonormal by the [Gram-Schmidt process](https://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process). Let $\Pi(a, b)$ be the projection from $a$ to $b$, namely $\Pi(a, b) = \frac{\langle a, b\rangle}{||b||} b$. Let
$$b_1 = a_1,$$ 
$$b_2 = a_2 - \Pi(a_2, b_1),$$
$$b_3 = a_3 - \Pi(a_3, b_1) - \Pi(a_3, b_2),$$
$$\dots$$
$$b_d = a_d - \sum_{i=1}^{d-1} \Pi(a_d, b_i).$$
Then $b_1, \dots, b_d$ are orthogonal, and the matrix $P = \begin{pmatrix} \frac{b_1}{||b_1||} & \dots & \frac{b_d}{||b_d||} \end{pmatrix}^T$ is orthonormal. Now we can generate the eigenvalues $\lambda_1, \dots, \lambda_d$ as we want, and let $\Lambda = diag(\lambda_1, \dots, \lambda_d)$. Then $Q = P \Lambda P^T$ is a real symmetric p.d. matrix with "good" eigenvalues (within our desired range).

The time complexity of Gram-Schmidt is $O(d^3)$, and the matrix multiplication is also $O(d^3)$. The total process takes $O(d^3)$ time. Code implements this method (using Eigen) is as follows:

```cpp
typedef Eigen::MatrixXd Matrix;

std::random_device rd;
std::mt19937 engine(rd());

// Project u to v
Matrix Proj(Matrix u, Matrix v) {
	auto a = u.transpose() * v;
	auto b = v.transpose() * v;
	double length = a(0, 0) / b(0, 0);
	return v * length;
}

// The Gram-Schmidt process
Matrix GramSchmidt(const Matrix& A) {
	int d = A.rows();
	Matrix a, P(d, d);
	for (int i = 0; i < d; ++i) {
		a = A(Eigen::all, i);
		for (int j = 0; j < i; ++j) {
			a -= Proj(A(Eigen::all, i), P(Eigen::all, j));
		}
		P(Eigen::all, i) = a;
	}
	for (int i = 0; i < d; ++i) {
		P(Eigen::all, i).normalize();
	}
	return P;
}

// Generate a real symmetric p.d. matrix with eigenvalues in [l, r]
Matrix GeneratePDMatrixWithinRange(int d, double l, double r) {
	std::uniform_real_distribution<double> entry(-1.0, 1.0), eigenval(l, r);
	Matrix A = Matrix::NullaryExpr(d, d, [&]() {return entry(engine);});
	Matrix P = GramSchmidt(A);
	Matrix D(d, d);
	D.setZero();
	for (int i = 0; i < d; ++i) {
		D(i, i) = eigenval(engine);
	}
	return P * D * P.transpose();
}
```

The figure below shows the distribution of the eigenvalues for the matrices generated using the above method with parameter $[l, r] = [0.5, 2.0]$.

![](/images/ellipsoids-generation-visualization/new_density.png)



## Visualization of Ellipses

Given a 2D ellipse characterized by 
$$x\in \mathbb{R}^d :\ (x - c)^T Q (x - c) \le 1,$$
where $Q\in \mathbb{R}^{2\times 2}$ and $c \in \mathbb{R}^2$, how can we visualize it using Matplotlib? Though is an [Ellipse patch](https://matplotlib.org/stable/api/_as_gen/matplotlib.patches.Ellipse.html) in the library, it specifies the ellipse by its width, height, and rotation angle. How do we visualize an ellipse using the matrix form?

Let $Q^{1/2}$ be the square root of $Q$. Recall that 
$$Q = P \Lambda P^T = P \sqrt{\Lambda} \sqrt{\Lambda} P^T = (P \sqrt{\Lambda} P^T)(P \sqrt{\Lambda} P^T) = Q^{1/2} Q^{1/2}.$$ 
The ellipse is equivalent to
$$x :\ ||Q^{1/2}(x - c)|| \le 1.$$
Let $y = Q^{1/2}(x - c)$, then we have
$$x :\ Q^{1/2}(x - c) = y,\ ||y|| \le 1.$$
Here $||y||\le 1$ is the unit disk. Rearranging the terms gives
$$x :\ x = Q^{-1/2}y + c,\ ||y|| \le 1.$$
In other words, let $f(y) = Q^{-1/2}y + c$, the ellipse can be constructed by transforming the unit disk $\mathcal{D}$ to $f(\mathcal{D})$.

_(Alternatively, one can also get $||\sqrt{\Lambda}P^T(x - c)||\le 1$ and $f'(y) = P\sqrt{\Lambda}^{-1}y + c$. Both $f$ and $f'$ produces the same ellipse, but the transformation of the unit disk is different.)_

To visualize the ellipse, we can sample points on the unit circle, and map them to the contour of the ellipse via $f$.

Code implements this method is as follows:

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.linalg import sqrtm

def draw_ellipse(ax, Q, c):
    A = np.linalg.inv(sqrtm(Q))
    ## Alternatively:
    # eigenvals, eigenvecs = np.linalg.eig(Q)
    # A = (1 / np.sqrt(eigenvals)) * eigenvecs
    theta = np.linspace(0, 2 * np.pi, 100)
    xs = (A @ [np.sin(theta), np.cos(theta)]).transpose() + c
    ax.plot(xs[:,0], xs[:,1])
```

A sample result is given below, where the green and red arrows illustrates the directions of the eigenvectors.

```python
Q = np.array(
    [[1.2360807e+00, -3.1592775e-01], 
    [-3.1592775e-01, 1.3596055e+00]])
c = np.array([2, 1])

fig, ax = plt.subplots()
ax.set_aspect(1)

draw_ellipse(ax, Q, c)
```

![](/images/ellipsoids-generation-visualization/ellipse.png)

The different between the transformations $f$ and $f'$ are visualized as below, where the green and red arrows illustrate the directions of the eigenvectors.

![](/images/ellipsoids-generation-visualization/transform.png)
