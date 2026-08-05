# Unscented Kalman Filter (UKF)
---

> [!NOTE]
> **DRAFT / AI-REFINED:** This content uses AI for structure and refinement, but still requires additional manual editing.

---

## 1. Overview & Motivation

The Extended Kalman Filter (EKF) linearizes non-linear models using a first-order Taylor series (Jacobians). This approach suffers from significant truncation errors in highly non-linear systems, which can lead to filter divergence. Furthermore, deriving analytical Jacobians for complex physics models is mathematically tedious and error-prone.

The Unscented Kalman Filter (UKF) eliminates the need for Jacobians entirely. Instead of approximating the *non-linear equations*, the UKF approximates the *probability distribution*. It uses the **Unscented Transform (UT)** to sample a minimal set of deterministic points (Sigma Points) around the mean. These points are propagated through the true non-linear functions, accurately capturing the posterior mean and covariance to the 3rd order for Gaussian distributions (compared to only 1st order for the EKF).

---

## 2. Non-Linear System Models

The system models remain identical to those used in the EKF, represented by non-linear differentiable functions $f$ and $h$:

* **Non-Linear Process Model:**

$$x_k = f(x_{k-1}, u_k) + w_k, \quad w_k \sim \mathcal{N}(0, Q)$$


* **Non-Linear Measurement Model:**

$$z_k = h(x_k) + v_k, \quad v_k \sim \mathcal{N}(0, R)$$



---

## 3. The Unscented Transform: Sigma Points & Weights

To represent a state $x$ with dimension $n$, mean $\hat{x}$, and covariance $P$, the UKF generates $2n + 1$ sigma points ($\mathcal{X}^{(i)}$).

### Scaling Parameters

* $\alpha$: Determines the spread of the sigma points around the mean (typically $10^{-4} \le \alpha \le 1$).
* $\kappa$: Secondary scaling parameter (usually set to $0$ or $3-n$).
* $\beta$: Incorporates prior knowledge of the distribution ($\beta = 2$ is optimal for Gaussian distributions).
* $\lambda$: Composite scaling parameter defined as $\lambda = \alpha^2(n + \kappa) - n$.

### Sigma Point Generation

The deterministic sigma points are generated using the matrix square root (typically implemented via **Cholesky Decomposition**):


$$\mathcal{X}^{(0)} = \hat{x}$$

$$\mathcal{X}^{(i)} = \hat{x} + \left( \sqrt{(n+\lambda)P} \right)_i \quad \text{for } i = 1 \dots n$$

$$\mathcal{X}^{(i)} = \hat{x} - \left( \sqrt{(n+\lambda)P} \right)_{i-n} \quad \text{for } i = n+1 \dots 2n$$


*(Note: $\left( \sqrt{M} \right)_i$ denotes the $i$-th column of the matrix square root).*

### Associated Weights

Each sigma point has a weight for calculating the mean ($W_m$) and a weight for the covariance ($W_c$):


$$W_m^{(0)} = \frac{\lambda}{n+\lambda}$$

$$W_c^{(0)} = \frac{\lambda}{n+\lambda} + (1 - \alpha^2 + \beta)$$

$$W_m^{(i)} = W_c^{(i)} = \frac{1}{2(n+\lambda)} \quad \text{for } i = 1 \dots 2n$$

---

## 4. UKF Recursive Algorithm

### Step 1: Predict (Time Update)

First, generate sigma points $\mathcal{X}_{k-1\vert{}k-1}$ from the previous state estimate $\hat{x}_{k-1\vert{}k-1}$ and covariance $P_{k-1\vert{}k-1}$.

* **Propagate Sigma Points through the Process Model:**

$$\mathcal{X}_{k\vert{}k-1}^{(i)} = f(\mathcal{X}_{k-1\vert{}k-1}^{(i)}, u_k)$$


* **Predicted (a priori) State Estimate:**

$$\hat{x}_{k\vert{}k-1} = \sum_{i=0}^{2n} W_m^{(i)} \mathcal{X}_{k\vert{}k-1}^{(i)}$$


* **Predicted (a priori) Error Covariance:**

$$P_{k\vert{}k-1} = \sum_{i=0}^{2n} W_c^{(i)} \left[ \mathcal{X}_{k\vert{}k-1}^{(i)} - \hat{x}_{k\vert{}k-1} \right] \left[ \mathcal{X}_{k\vert{}k-1}^{(i)} - \hat{x}_{k\vert{}k-1} \right]^T + Q$$



### Step 2: Update (Measurement Update)

To capture the new distribution spread, **redraw** the sigma points $\mathcal{X}_{k\vert{}k-1}$ using the newly predicted mean $\hat{x}_{k\vert{}k-1}$ and covariance $P_{k\vert{}k-1}$.

* **Propagate Sigma Points through the Measurement Model:**

$$\mathcal{Z}_{k\vert{}k-1}^{(i)} = h(\mathcal{X}_{k\vert{}k-1}^{(i)})$$


* **Predicted Measurement Mean:**

$$\hat{z}_k = \sum_{i=0}^{2n} W_m^{(i)} \mathcal{Z}_{k\vert{}k-1}^{(i)}$$


* **Innovation Covariance:**

$$S_k = \sum_{i=0}^{2n} W_c^{(i)} \left[ \mathcal{Z}_{k\vert{}k-1}^{(i)} - \hat{z}_k \right] \left[ \mathcal{Z}_{k\vert{}k-1}^{(i)} - \hat{z}_k \right]^T + R$$


* **Cross-Covariance Matrix (State-Measurement correlation):**

$$P_{xz} = \sum_{i=0}^{2n} W_c^{(i)} \left[ \mathcal{X}_{k\vert{}k-1}^{(i)} - \hat{x}_{k\vert{}k-1} \right] \left[ \mathcal{Z}_{k\vert{}k-1}^{(i)} - \hat{z}_k \right]^T$$


* **Optimal Kalman Gain:**

$$K_k = P_{xz} S_k^{-1}$$


* **Updated (a posteriori) State Estimate:**

$$\hat{x}_{k\vert{}k} = \hat{x}_{k\vert{}k-1} + K_k(z_k - \hat{z}_k)$$


* **Updated (a posteriori) Error Covariance:**

$$P_{k\vert{}k} = P_{k\vert{}k-1} - K_k S_k K_k^T$$



---

## 5. Production Engineering Considerations

1. **Cholesky Decomposition Failure:** The matrix $(n+\lambda)P$ must be positive definite to compute the matrix square root via Cholesky decomposition. Numerical instability during the covariance update ($P_{k\vert{}k}$) can cause the matrix to lose positive-definiteness, crashing the algorithm. Implement numerical safe-guards, such as forcing symmetry ($P = \frac{P + P^T}{2}$) and adding a small jitter matrix ($\epsilon I$) if the decomposition fails.
2. **Computational Cost:** The UKF requires passing $2n+1$ points through the non-linear models. If $n$ (state dimension) is extremely large, the computational overhead can exceed that of the EKF. However, because Jacobians are not required, the UKF is highly modular—you can swap out the functions $f$ and $h$ without rewriting any calculus.
3. **Augmented UKF vs. Standard UKF:** The formulation above assumes additive process and measurement noise. If noise is non-additive (e.g., noise that scales multiplicatively with the state), the state vector must be **augmented** to include the noise variables directly before generating the sigma points, increasing the dimensionality of the UT.