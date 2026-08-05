# cpt_bringup

This is package is just a place to bring to gether all the steps into a single launch file, from launching the simulation to the controller to optionally the visualization tools too.

## Package Structure & File Descriptions

This package handles the core trajectory planning for the quadrotor swarm. 
```
📂 `cpt_bringup/`
└── 📂 `launch/`
    └── `three_drone_launch.py`


```
### File Breakdown

**`launch/three_drone_launch.py`**
This is the main entry point for the simulation. It spawns three quadrotor instances of the X500 drone into the Gazebo environment, remaps their individual namespaces (e.g., `drone_1`, `drone_2`, `drone_3`), and bridges the PX4 SITL physics with the ROS 2 network. And then it runs the simple control algorithms to send the same trajectory (offseted, withrespect to their starting state) so that it appears they are working together.



