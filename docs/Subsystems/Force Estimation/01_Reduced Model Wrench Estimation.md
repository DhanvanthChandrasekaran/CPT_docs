# Reduced Model Wrench Estimation

> ⚠️ **Note:** This documentation is the **not final draft** and is AI-generated. It provides a detailed explanation based on the mathematical models and theory presented in the research paper *"Robust Collaborative Object Transportation Using Multiple MAVs"* (arXiv:1711.08753v1) by Andrea Tagliabue, Mina Kamel, Roland Siegwart, and Juan Nieto[cite: 1].

## 1. Introduction and Theory
The **Reduced Model Wrench Estimation** framework is designed to estimate the external forces and torques (the "wrench") acting on a Micro Aerial Vehicle (MAV) during physical interactions or environmental disturbances. The defining characteristic of the reduced model is that it assumes the closed-loop attitude dynamics of the MAV can be approximated as a decoupled, second-order linear system parameterized by Euler angles. 

This strategy is highly appealing because many commercial MAVs possess proprietary low-level attitude controllers and do not provide direct feedback of individual motor speeds. By abstracting the complex nonlinear aerodynamic and rotor dynamics into a simplified, aggregated response to commanded thrust and attitude inputs, we can construct an efficient Extended Kalman Filter (EKF) to continuously estimate the unmeasured external wrenches.

## 2. Reference Frames and Variable Definitions

To express the dynamics accurately, we define the following reference frames and physical quantities:

### 2.1 Reference Frames
*   $I$: The **Inertial reference frame**, with its positive $z$-axis pointing upwards, opposing gravity.
*   $B$: The **Body reference frame**, firmly attached to the Center of Gravity (CoG) of the MAV.

### 2.2 Variables and Parameters
*   ${}_I p \in \mathbb{R}^{3 \times 1}$: Position vector of the MAV expressed in $I$ $[m]$.
*   ${}_I v \in \mathbb{R}^{3 \times 1}$: Linear velocity vector of the MAV expressed in $I$ $[m \cdot s^{-1}]$.
*   $\eta = [\phi, \theta, \psi]^T \in \mathbb{R}^{3 \times 1}$: Roll, pitch, and yaw Euler angles $[rad]$.
*   $R_{IB}(\eta) \in \mathbb{R}^{3 \times 3}$: Orthogonal rotation matrix transforming vectors from $B$ to $I$.
*   $m \in \mathbb{R}$: Total mass of the MAV $[kg]$.
*   $g \in \mathbb{R}$: Scalar gravitational acceleration $[m \cdot s^{-2}]$.
*   $K_{drag} \in \mathbb{R}^{3 \times 3}$: Positive diagonal matrix representing linear aerodynamic drag coefficients $[N \cdot s \cdot m^{-1}]$.
*   $U_4 \in \mathbb{R}$: Total scalar thrust produced by the propellers along the $z$-axis of $B$ $[N]$.
*   ${}_I F^{ext} \in \mathbb{R}^{3 \times 1}$: Vector of unknown external forces acting on the MAV, expressed in $I$ $[N]$.
*   ${}_B M^{ext} \in \mathbb{R}^{3 \times 1}$: Vector of unknown external torques acting on the MAV, expressed in $B$ $[N \cdot m]$.
*   $\omega_{n,\phi}, k_\phi, \xi_\phi \in \mathbb{R}$: Positive constants characterizing the closed-loop roll dynamic response (natural frequency $[rad \cdot s^{-1}]$, static gain, and damping ratio). Similar parameters exist for pitch and yaw.
*   $J_{xx}, J_{yy}, J_{zz} \in \mathbb{R}$: Diagonal entries of the MAV's moment of inertia tensor $[kg \cdot m^2]$.

## 3. System Modeling

### 3.1 Translational Dynamics
The translational motion is modeled by balancing the forces acting on the MAV body: (1) Actuator Thrust, (2) Gravity, (3) Aerodynamic Drag, and (4) External Contact Forces.

The kinematic and dynamic differential equations are:
$$ {}_I \dot{p} = {}_I v $$
$$ {}_I \dot{v} = \frac{1}{m} R_{IB}(\eta) \left( \begin{bmatrix} 0 \\ 0 \\ U_4 \end{bmatrix} - K_{drag} {}_B v \right) - \begin{bmatrix} 0 \\ 0 \\ g \end{bmatrix} + \frac{1}{m} {}_I F^{ext} $$

*Explanation:* The thrust vector $\begin{bmatrix} 0, 0, U_4 \end{bmatrix}^T$ is generated purely along the body $z$-axis. The drag is assumed to be linearly proportional to the body velocity ${}_B v$. The external force is added directly to the sum of forces and scaled by the inverse of the mass to yield linear acceleration.

### 3.2 Rotational Dynamics
Since the actual motor commands are abstracted, the MAV's attitude response to high-level commands ($\phi_{cmd}, \theta_{cmd}, \psi_{cmd}$) is modeled as a decoupled second-order system. For example, the rotational dynamics around the body $x$-axis (roll) are approximated as:
$$ \ddot{\phi} = \omega_{n,\phi}^2 (k_\phi \phi_{cmd} - \phi) - 2 \xi_\phi \omega_{n,\phi} \dot{\phi} + \frac{{}_B M_x^{ext}}{J_{xx}} $$

*Explanation:* This equation mimics a damped spring-mass system driven by an attitude command $\phi_{cmd}$. The external torque ${}_B M_x^{ext}$ acts as an unknown disturbance accelerating the rotation. The identical structure applies to pitch ($\theta$) and yaw ($\psi$) dynamics.

## 4. Filter Formulation (Extended Kalman Filter)

An Extended Kalman Filter (EKF) is synthesized to estimate the external wrench online. 

### 4.1 State, Input, and Measurement Vectors
Since the wrench dynamics are arbitrary, they are modeled as random walks driven by zero-mean white Gaussian noise. The system state is augmented to include these unknown variables.

*   **State Vector** $x \in \mathbb{R}^{18 \times 1}$:
    $$ x = \begin{bmatrix} {}_I p^T & {}_I v^T & \eta^T & \dot{\eta}^T & {}_I F^{ext,T} & {}_B M^{ext,T} \end{bmatrix}^T $$
*   **Input Vector** $u \in \mathbb{R}^{4 \times 1}$:
    $$ u = \begin{bmatrix} \phi_{cmd} & \theta_{cmd} & \psi_{cmd} & U_4 \end{bmatrix}^T $$
    *(Note: Depending on the controller, $U_4$ can also be expressed as $F_{cmd}$).*
*   **Measurement Vector** $z \in \mathbb{R}^{6 \times 1}$:
    Assuming a Motion Capture System or onboard visual-inertial odometry provides direct pose measurements:
    $$ z = \begin{bmatrix} {}_I p^T & \eta^T \end{bmatrix}^T $$

### 4.2 Estimation Steps
1.  **Prediction:** The EKF propagates the 18-dimensional state vector forward in time using the non-linear process models described in Section 3. The external forces and torques are propagated as constants (${}_I \dot{F}^{ext} = 0$, ${}_B \dot{M}^{ext} = 0$) subjected only to process noise covariance $Q$.
2.  **Jacobian Calculation:** The EKF linearizes the continuous-time dynamics around the current state estimate to propagate the state covariance matrix $P$.
3.  **Measurement Update:** Upon receiving pose measurements $z$, the Kalman Gain is computed, the state $x$ is corrected, and the covariance $P$ is updated. Through the dynamic coupling in the process model, errors in pose tracking actively correct the estimates of the unmeasured external forces and torques.
