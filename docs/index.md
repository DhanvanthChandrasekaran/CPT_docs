# Collaborative Payload Transport (CPT) using Multiple Quadrotors

[![Ubuntu 22.04](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![ROS 2 - Humble](https://img.shields.io/badge/ROS_2-Humble-blue?logo=ros&logoColor=white)](https://docs.ros.org/en/humble/)
[![PX4 - Autopilot](https://img.shields.io/badge/PX4-Autopilot-102A4A?logo=px4&logoColor=white)](https://px4.io/)
[![Gazebo - Garden](https://img.shields.io/badge/Gazebo-Garden-orange?logo=gazebo&logoColor=white)](https://gazebosim.org/home)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)

> **⚠️ WORK IN PROGRESS**
> Currently, only the draft documentation for a portion **Subsystems** is available. Documentation for the remaining Sections is actively being written and will be added in future updates.

Welcome to the documentation for the Collaborative Payload Transport (CPT) project, developed under the guidance of Professor Krishna Prakash Yadav at the National Institute of Technology (NIT) Warangal.

This project focuses on the challenge of coordinating multiple quadrotors to synchronously lift and transport a shared heavy payload.

---

## Documentation Structure

Whether you are looking to understand our control mathematics, replicate our simulation, or deploy the code, the documentation is split into three main areas:

*   **[The Code](./Code)** 
    Deep dive into the custom ROS 2 nodes, explaining the logic, structure, and reasoning behind our control and perception algorithms.
*   **[The Subsystems](./Subsystems)** 
    Explore the theory, implementation details, and selection considerations for our hardware and software subsystems.
*   **[The Simulation](./Simulation)** 
    A comprehensive guide to our Gazebo Garden setup, detailing the hurdles we faced, how to launch the environment, and a guide for making future modifications.

---

## System Architecture

Our stack is designed to be easily reproducible and containerized for portability, utilizing standard tools in the modern robotics ecosystem.

### Software Environment
| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **OS** | Ubuntu 22.04 | Primary operating system |
| **Middleware** | ROS 2 Humble | High-level control and perception logic |
| **Firmware** | PX4 | Low-level flight control and stabilization |
| **Simulation** | Gazebo Garden | Physics and environment simulation |
| **Deployment** | Docker | Codebase containerization for easy portability |

### Hardware Stack
| Component | Hardware |
| :--- | :--- |
| **Compute Module** | Jetson Orin Nano Super |
| **Vision & Perception** | Intel RealSense D435if |
| **Flight Controller** | Pixhawk 6c |

---

## Getting Started
*TODO*