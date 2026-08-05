# Linear Kalman Filter (KF)

---

> [!NOTE]
> **DRAFT / AI-REFINED:** This content uses AI for structure and refinement, but still requires additional manual editing.

---

## 1. Overview & Core Assumptions

The linear discrete-time Kalman Filter is an optimal recursive estimator that minimizes the mean squared error of the state estimate for linear dynamical systems. To guarantee optimality, the filter relies on strict mathematical and statistical foundations:

1. **System Linearity:** The evolution of the system state and the measurement mapping must be linear functions (or linearized via an Extended Kalman Filter).
2. **Gaussian Noise:** Both the process noise ($w_k$) and measurement noise ($v_k$) are assumed to be independent, white, zero-mean Gaussian random processes.
3. **Uncorrelated Noise:** The process and measurement noise sequences are mutually uncorrelated across all time steps ($\mathbb{E}[w_k v_j^T] = 0$ for all $k, j$).

---

## 2. Ground-Truth System & Measurement Models

Before defining the estimator, we formally establish the true physical system model. The true state evolution and sensor observations are governed by:

* **Process Model:**

$$x_k = F x_{k-1} + B u_k + w_k, \quad w_k \sim \mathcal{N}(0, Q)$$


* **Measurement Model:**

$$z_k = H x_k + v_k, \quad v_k \sim \mathcal{N}(0, R)$$



In reality, physics models (such as rigid-body kinematics or ballistic equations) capture the nominal system behavior, but real-world environments are stochastic. External unmodeled disturbances (e.g., aerodynamic turbulence, micro-gusts, unmodeled friction) act on the system as process noise $w_k$. Sensors are similarly imperfect, introducing measurement noise $v_k$. The Kalman Filter uses observations to continuously bound the cumulative drift caused by these uncertainties.

---

## 3. Symbol & Dimension Dictionary

Let $n$ represent the dimension of the state vector, $m$ represent the dimension of the measurement vector, and $c$ represent the dimension of the control input vector.

| Symbol | Description | Dimensions (Rows $\times$ Cols) |
| --- | --- | --- |
| **Indices & Operators** |  |  |
| $k$ | Current discrete time step. | Scalar |
| $k-1$ | Previous discrete time step. | Scalar |
| $\hat{x}_{k\Vert{}k-1}$ | A priori state estimate (predicted state at time $k$ given data up to $k-1$). | $n \times 1$ |
| $\hat{x}_{k\Vert{}k}$ | A posteriori state estimate (updated state at time $k$ incorporating measurement $z_k$). | $n \times 1$ |
| $^T$ | Matrix transpose operator. | - |
| $^{-1}$ | Matrix inverse operator. | - |
| **State & Covariance** |  |  |
| $x_k$ | True state vector (e.g., position, velocity, orientation). | $n \times 1$ |
| $P_{k\Vert{}k-1}$ | A priori estimate error covariance matrix; uncertainty of the prediction. | $n \times n$ |
| $P_{k\Vert{}k}$ | A posteriori estimate error covariance matrix; uncertainty of the updated estimate. | $n \times n$ |
| **System Matrices** |  |  |
| $F$ | State transition matrix; maps the state from time $k-1$ to $k$. | $n \times n$ |
| $B$ | Control input matrix; maps control inputs to state variations. | $n \times c$ |
| $u_k$ | Control input vector; known deterministic commands applied to the system. | $c \times 1$ |
| $H$ | Observation matrix; maps the state space into the measurement space. | $m \times n$ |
| **Noise Covariances** |  |  |
| $Q$ | Process noise covariance matrix; models uncertainty in system dynamics. | $n \times n$ |
| $R$ | Measurement noise covariance matrix; models sensor uncertainty/variance. | $m \times m$ |
| **Update Variables** |  |  |
| $y_k$ | Innovation (measurement residual) vector; discrepancy between actual and predicted measurement. | $m \times 1$ |
| $S_k$ | Innovation covariance matrix; total uncertainty of the residual. | $m \times m$ |
| $K_k$ | Optimal Kalman Gain matrix; blending weight balancing model and sensor trust. | $n \times m$ |
| $I$ | Identity matrix. | $n \times n$ |

---

## 4. Recursive Filter Algorithm

### Step 1: Predict (Time Update)

The time update projects the current state and error covariance forward in time using the deterministic system dynamics and process noise statistics, independent of incoming sensor data.

* **Predicted (a priori) State Estimate:**

$$\hat{x}_{k\vert{}k-1} = F \hat{x}_{k-1\vert{}k-1} + B u_k$$


* **Predicted (a priori) Error Covariance:**

$$P_{k\vert{}k-1} = F P_{k-1\vert{}k-1} F^T + Q$$



### Step 2: Update (Measurement Update)

The measurement update incorporates the latest sensor observation to correct the predicted state estimate, weighting the correction using the optimal Kalman Gain.

* **Innovation (Residual):**

$$y_k = z_k - H \hat{x}_{k\vert{}k-1}$$


* **Innovation Covariance:**

$$S_k = H P_{k\vert{}k-1} H^T + R$$


* **Optimal Kalman Gain:**

$$K_k = P_{k\vert{}k-1} H^T S_k^{-1}$$


* **Updated (a posteriori) State Estimate:**

$$\hat{x}_{k\vert{}k} = \hat{x}_{k\vert{}k-1} + K_k y_k$$


* **Updated (a posteriori) Error Covariance:**

$$P_{k\vert{}k} = (I - K_k H) P_{k\vert{}k-1}$$



---

## 5. Production Engineering Considerations

* **Numerical Stability (Joseph Form):** In embedded systems and long-horizon software implementations, the standard covariance update equation $P_{k\vert{}k} = (I - K_k H) P_{k\vert{}k-1}$ is susceptible to round-off errors, which can destroy matrix symmetry and positive-definiteness, eventually causing filter divergence. For safety-critical systems, implement the numerically stable **Joseph form**:

$$P_{k\vert{}k} = (I - K_k H) P_{k\vert{}k-1} (I - K_k H)^T + K_k R K_k^T$$


* **Covariance Tuning:** Matrix tuning parameters $Q$ and $R$ dictate the filter's transient response and steady-state error. Over-tuning $Q$ makes the filter overly sensitive to high-frequency sensor noise, while under-tuning $Q$ causes the filter to lag behind rapid system maneuvers.