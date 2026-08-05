# Extended Kalman Filter (EKF)

---

> [!NOTE]
> **DRAFT / AI-REFINED:** This content uses AI for structure and refinement, but still requires additional manual editing.

---

## 1. Overview & Motivation

The standard Kalman Filter (KF) is strictly optimal only for linear systems. However, real-world kinematics and sensor models—such as resolving 3D orientation (Euler angles/quaternions) or mapping visual/LIDAR features to a global coordinate frame—are inherently non-linear.

The Extended Kalman Filter (EKF) overcomes this by locally linearizing the non-linear process and measurement models around the current mean estimate using a first-order Taylor series expansion. This linearization is achieved by computing **Jacobian matrices** at every time step.

---

## 2. Non-Linear System Models

Unlike the linear KF, which uses constant matrices $F$ and $H$, the EKF utilizes non-linear differentiable functions $f$ and $h$ to describe state evolution and sensor observations:

* **Non-Linear Process Model:**

$$x_k = f(x_{k-1}, u_k) + w_k, \quad w_k \sim \mathcal{N}(0, Q)$$



*Here, $f$ computes the predicted state directly from the previous state and control input.*
* **Non-Linear Measurement Model:**

$$z_k = h(x_k) + v_k, \quad v_k \sim \mathcal{N}(0, R)$$



*Here, $h$ maps the predicted true state space into the expected measurement space.*

---

## 3. The Jacobian Matrices (Linearization)

To propagate the error covariance and compute the Kalman Gain, we must linearize $f$ and $h$. We do this by computing the partial derivatives (Jacobians) of these functions evaluated at the most recent state estimate.

**State Transition Jacobian ($F_k$):**
Evaluated at the a posteriori state estimate from the previous time step.


$$F_k = \left. \frac{\partial f}{\partial x} \right\vert{}_{\hat{x}_{k-1\vert{}k-1}, u_k}$$

**Observation Jacobian ($H_k$):**
Evaluated at the a priori state estimate of the current time step.


$$H_k = \left. \frac{\partial h}{\partial x} \right\vert{}_{\hat{x}_{k\vert{}k-1}}$$

---

## 4. Symbol & Dimension Dictionary

Let $n$ represent the dimension of the state vector, $m$ represent the dimension of the measurement vector, and $c$ represent the dimension of the control input vector.

| Symbol | Description | Dimensions (Rows $\times$ Cols) |
| --- | --- | --- |
| $f$ | Non-linear state transition function. | Vector mapping $\mathbb{R}^{n+c} \to \mathbb{R}^n$ |
| $h$ | Non-linear observation (measurement) function. | Vector mapping $\mathbb{R}^n \to \mathbb{R}^m$ |
| $F_k$ | State transition Jacobian matrix at time $k$. | $n \times n$ |
| $H_k$ | Observation Jacobian matrix at time $k$. | $m \times n$ |
| $Q$ | Process noise covariance matrix. | $n \times n$ |
| $R$ | Measurement noise covariance matrix. | $m \times m$ |
| $\hat{x}_{k\Vert{}k-1}$ | A priori state estimate. | $n \times 1$ |
| $\hat{x}_{k\Vert{}k}$ | A posteriori state estimate. | $n \times 1$ |
| $P_{k\Vert{}k-1}$ | A priori estimate error covariance. | $n \times n$ |
| $P_{k\Vert{}k}$ | A posteriori estimate error covariance. | $n \times n$ |

---

## 5. EKF Recursive Algorithm

### Step 1: Predict (Time Update)

The state is predicted using the **full non-linear function** $f$, while the covariance is propagated using the **linearized Jacobian** $F_k$.

* **Predicted (a priori) State Estimate:**

$$\hat{x}_{k\vert{}k-1} = f(\hat{x}_{k-1\vert{}k-1}, u_k)$$


* **Predicted (a priori) Error Covariance:**

$$P_{k\vert{}k-1} = F_k P_{k-1\vert{}k-1} F_k^T + Q$$



### Step 2: Update (Measurement Update)

The expected measurement is generated using the **full non-linear function** $h$, while the Kalman Gain and updated covariance rely on the **linearized Jacobian** $H_k$.

* **Innovation (Residual):**

$$y_k = z_k - h(\hat{x}_{k\vert{}k-1})$$


* **Innovation Covariance:**

$$S_k = H_k P_{k\vert{}k-1} H_k^T + R$$


* **Optimal Kalman Gain:**

$$K_k = P_{k\vert{}k-1} H_k^T S_k^{-1}$$


* **Updated (a posteriori) State Estimate:**

$$\hat{x}_{k\vert{}k} = \hat{x}_{k\vert{}k-1} + K_k y_k$$


* **Updated (a posteriori) Error Covariance (Standard Form):**

$$P_{k\vert{}k} = (I - K_k H_k) P_{k\vert{}k-1}$$



*(Note: The Joseph form $P_{k\vert{}k} = (I - K_k H_k) P_{k\vert{}k-1} (I - K_k H_k)^T + K_k R K_k^T$ is highly recommended for production deployment to maintain numerical stability).*

---

## 6. Production Engineering Considerations

1. **Truncation Errors:** Because the EKF relies on a first-order Taylor approximation, highly non-linear dynamics can cause large truncation errors. If the true system deviates significantly from the tangent plane defined by the Jacobian, the filter may diverge. For highly non-linear systems, the Unscented Kalman Filter (UKF) is often a safer alternative.
2. **Jacobian Computation Overhead:** Calculating Jacobians analytically (via calculus) is mathematically exact but computationally expensive and prone to human derivation errors. Numerical differentiation (e.g., finite difference methods or automatic differentiation libraries) can be used, though it introduces a slight performance overhead.
3. **Initialization Sensitivity:** Unlike the linear KF, the EKF is extremely sensitive to initial conditions. A poor initial state estimate ($\hat{x}_{0\vert{}0}$) will cause the first Jacobians to be evaluated at the wrong operating point, leading to immediate divergence.