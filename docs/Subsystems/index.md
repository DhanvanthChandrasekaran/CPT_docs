# The Subsystems

This section of the documentation covers the implementation, selection considerations, and theoretical background behind each of the core subsystems in the Collaborative Payload Transport (CPT) project.

> **⚠️ WORK IN PROGRESS**
> Currently, only the draft documentation for **State Estimation** and **Force Estimation** is available. Documentation for the remaining hardware and software subsystems is actively being written and will be added in future updates.

## Available Documentation

Please navigate to the respective pages below for detailed information on our current subsystems:

*   **[State Estimation](./State Estimation (VIO)/01_theory_and_algorithms.md)** 
    *Covers Visual-Inertial Odometry (VIO) using the RealSense D435if, Orin Nano processing, and PX4 EKF2 fusion.*
*   **[Force Estimation](./Force Estimation/01_Reduced Model Wrench Estimation.md)** 
    *Covers the dynamic/disturbance observer approach to computationally estimate payload tether tension without physical load cells.*

---
*Check back later for updates regarding hardware selection (Jetson Orin Nano, Pixhawk 6c) and lower-level control implementations.*