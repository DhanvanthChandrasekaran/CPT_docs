# VIO Implementation & Calibration

---

> [!NOTE]
> **DRAFT / AI-REFINED:** This content uses AI for structure and refinement, but still requires additional manual editing.

---
## Available Variants of VIO Implementations

The open-source robotics community has produced several highly robust VIO frameworks. They generally fall into one of two camps—Filtering or Optimization—and handle image data either via Features (extracting descriptors) or Direct methods (tracking raw pixel intensities). 

Below is a breakdown of the most prominent implementations available today.

### 1. OpenVINS
*   **Source:** Robot Perception and Navigation Group (RPNG) at the University of Delaware.
*   **Approach:** Filtering-based (Multi-State Constraint Kalman Filter / MSCKF).
*   **Unique Features:** OpenVINS is an extremely lightweight, on-manifold Extended Kalman Filter (EKF). Because it uses the MSCKF formulation, it achieves $O(n)$ complexity, making it computationally cheap while maintaining high accuracy. A major standout feature is its modular state system, which allows for out-of-the-box online spatial and temporal calibration (it can calculate the camera-IMU offsets on the fly if they aren't perfectly known). It scales to an arbitrary number of cameras and natively supports both ROS 1 and ROS 2, making it highly attractive for modern, compute-constrained drone platforms.

### 2. VINS-Mono & VINS-Fusion
*   **Source:** Aerial Robotics Group at the Hong Kong University of Science and Technology (HKUST).
*   **Approach:** Optimization-based (Sliding Window Bundle Adjustment / Factor Graph).
*   **Unique Features:** VINS-Mono is widely considered the industry baseline for optimization-based VIO. It tightly couples visual and inertial measurements by utilizing **IMU pre-integration** on the $SO(3)$ manifold to constrain the optimization solver (usually Ceres Solver). It includes a robust loop closure module (using DBoW2 bag-of-words) to eliminate long-term drift. **VINS-Fusion** is the successor, expanding the architecture to support stereo cameras, stereo+IMU, and even GPS fusion. While highly accurate, the non-linear optimization demands significant CPU resources, which can bottleneck edge computers.

### 3. ROVIO (Robust Visual Inertial Odometry)
*   **Source:** Autonomous Systems Lab (ASL) at ETH Zurich.
*   **Approach:** Direct, Filtering-based (EKF).
*   **Unique Features:** Unlike most EKFs that extract abstract corners (like ORB or SIFT), ROVIO operates directly on pixel intensity patches. It tracks the photometric error of these patches directly within the filter's state vector. By bypassing traditional feature descriptors, ROVIO is extraordinarily robust to extreme motion blur and poorly textured environments—common scenarios for aggressive drone flight. It is highly efficient but functions strictly as a monocular system and does not support loop closure.

### 4. SVO 2.0 (Semi-Direct Visual Odometry)
*   **Source:** Robotics and Perception Group (RPG) at the University of Zurich (UZH).
*   **Approach:** Semi-Direct Optimization.
*   **Unique Features:** SVO bridges the gap between direct and feature-based methods. It uses *direct* pixel intensity alignment for ultra-fast frame-to-frame motion estimation, but relies on *feature-based* methods for the joint optimization of structure and motion. This architecture decouples the heavy feature extraction from the tracking thread, allowing SVO to run at blinding speeds (up to 400 Hz on standard laptops or easily real-time on embedded processors). It is heavily utilized in micro-aerial vehicles (MAVs) where high-frequency pose updates and minimal CPU overhead are critical.

### 5. ORB-SLAM3
*   **Source:** University of Zaragoza.
*   **Approach:** Optimization-based (Factor Graph with Maximum A Posteriori estimation).
*   **Unique Features:** ORB-SLAM3 is an absolute powerhouse. It is a purely feature-based system (using ORB descriptors) that supports monocular, stereo, and RGB-D cameras with or without an IMU. Its standout feature is its **multi-map system**: if the camera is blinded or tracking is completely lost, it simply starts a new map. When it recognizes a previous location, it seamlessly merges the maps together and optimizes the entire trajectory. It provides state-of-the-art accuracy but requires substantial memory and compute.

### 6. OKVIS (Open Keyframe-based Visual-Inertial SLAM)
*   **Source:** Imperial College London / ETH Zurich.
*   **Approach:** Optimization-based (Keyframe Sliding Window).
*   **Unique Features:** OKVIS is one of the earliest pioneers of tight visual-inertial non-linear optimization. It tightly couples visual reprojection errors (using BRISK descriptors) and IMU error terms into a single cost function. While slightly older than VINS or ORB-SLAM3, its architecture heavily influenced modern optimization-based systems and remains a highly reliable benchmark for keyframe-based VIO.

### 7. NVIDIA Isaac ROS Visual SLAM (cuVSLAM)
*   **Source:** NVIDIA.
*   **Approach:** GPU-Accelerated Optimization-based (Pose Graph).
*   **Unique Features:** Built on top of the proprietary `cuVSLAM` library, this is a highly optimized, hardware-accelerated package designed specifically for NVIDIA GPUs and Jetson platforms (like the Orin series). It shifts the heavy lifting of feature extraction, matching, and non-linear least squares optimization (via `cuNLS`) directly to the GPU. It maintains a pose graph for landmarks and automatically triggers loop closures. One of its most striking features is its scalability: it natively supports up to 32 cameras (16 stereo pairs) simultaneously, drastically improving robustness in featureless environments. Furthermore, it handles parameter tuning entirely under the hood, adapting to the environment without requiring manual algorithmic adjustments.

---

## Framework Selection
There are several popular variants of VIO implementations available, including ROVIO, VINS-Mono, VINS-Fusion, and OpenVINS. 

**Our Choice: OpenVINS**
We selected OpenVINS for this project due to two main factors:

1.  **Computational Efficiency:** It has a significantly lower computational cost during runtime compared to optimization-heavy frameworks like VINS-Mono.
2.  **Software Ecosystem:** Our software framework is ROS 2. OpenVINS natively supports both ROS 1 and ROS 2, whereas many legacy VIO systems rely on ROS 1 (which reached End-of-Life in May 2025).

---

## Calibration Process Overview
Precise calibration of the camera and IMU is required for accurate state estimation.

### Camera Calibration (Intrinsics)
Camera calibration finds the physical parameters (like focal length) and distortion parameters. Different camera models are best suited for different types of lenses. Because our system uses a stereo depth camera, both lenses were calibrated.

### Camera-IMU Calibration (Extrinsics)
This process finds the exact relative position and orientation of the IMU and the cameras. Without this, slight physical offsets between the sensors will introduce errors in the data fusion that the algorithm cannot account for. 
*   *Implementation:* Both camera intrinsics and camera-IMU extrinsics were calibrated using the **Kalibr** tool. *(TODO: Elaborate on exact Kalibr execution steps).*

### IMU Calibration (Intrinsics)
While it is possible to use the manufacturer's default intrinsic parameters for the IMU, a manual calibration yields optimal performance. 
*   *Methodology:* This involves collecting a static IMU recording for at least 12 hours at the operating frequency. This data is run through **Allan Variance** to find the exact noise parameters (random walk, bias instability) of the sensor. 
*   *Importance:* Understanding this uncertainty tells the VIO algorithm exactly how much it should "trust" the IMU predictions versus the visual updates.
*   *Current Status:* A rigorous Allan Variance calibration is pending. Currently, we are using the manufacturer's provided IMU parameters, and tracking has remained highly solid.

---

## Calibration Process (Kalibr)

Precise calibration of the camera and IMU is required for accurate state estimation. We use the **Kalibr visual-inertial calibration toolbox**, which solves for the intrinsic parameters, the spatial transformations (extrinsics), and the temporal offsets between the sensors. The workflow is executed in four sequential phases:

### 1. Preparation and Prerequisites
*   **Calibration Target:** We use an **AprilGrid** instead of a standard checkerboard. AprilGrids are robust to partial occlusions, meaning the tracking won't fail if the camera only sees a portion of the board during tight rotational movements.
*   **Target Configuration:** A `target.yaml` file is defined containing the physical dimensions of the tags and the exact spacing between them. 
*   **Data Collection:** All sensor data is recorded into ROS `.bag` files. It is critical that the stereo images are recorded in grayscale and that the IMU publishes raw, unfiltered acceleration and angular velocity data at its maximum frequency.

### 2. IMU Intrinsics (Allan Variance)
Before calibrating the cameras, we must define the IMU's underlying noise characteristics.
*   **Data Collection:** The sensor rig is left completely stationary for an extended period (typically 2 to 12 hours) while recording the IMU topic. 
*   **Processing:** The static data is processed using an Allan Variance tool (e.g., `imu_utils` or `allan_variance_ros`). 
*   **Output:** This generates an `imu.yaml` file containing the **White Noise Density** and **Random Walk** values for both the accelerometer and gyroscope. These parameters dictate how much OpenVINS will trust the IMU predictions between visual updates.

### 3. Stereo Camera Intrinsics & Extrinsics
This step calculates the focal lengths, principal points, and distortion parameters for both cameras, alongside the exact 3D transformation (baseline) between the left and right lenses.
*   **Data Collection:** We record a ROS bag while slowly moving the stereo camera in front of the AprilGrid. The motion must be smooth to avoid motion blur, ensuring the grid is viewed from multiple angles, distances, and orientations.
*   **Execution:** We run the `kalibr_calibrate_cameras` tool, passing in the bag file, the ROS topics for both cameras, the camera model (e.g., `pinhole-radtan`), and the `target.yaml`.
*   **Output:** Kalibr minimizes the reprojection error and outputs a `camchain.yaml` file containing the intrinsics for both cameras and their relative spatial relationship.

### 4. Camera-IMU Spatiotemporal Extrinsics 
This final step calculates the rigid 6-DOF transformation between the IMU and the camera system, as well as the exact time delay (temporal offset) between their data streams.
*   **Data Collection:** A new ROS bag is recorded. This time, the motion must aggressively excite all 3 axes of the IMU (both translation and rotation) while keeping the AprilGrid in the camera's field of view. This requires a careful physical balance: moving fast enough to register strong IMU readings, but smooth enough to avoid heavy camera motion blur.
*   **Execution:** We run the `kalibr_calibrate_imu_camera` tool. This requires all previously generated files: the dynamic bag file, the `camchain.yaml`, the `imu.yaml`, and the `target.yaml`.
*   **Output:** Kalibr performs a batch optimization using continuous-time splines to align the IMU motion with the visual motion. It generates a comprehensive `camchain-imucam.yaml` containing the final extrinsics and time offsets, which is directly ingested by OpenVINS.