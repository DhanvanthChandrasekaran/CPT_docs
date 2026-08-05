# Hardware Considerations for VIO

---

> [!NOTE]
> **DRAFT / AI-REFINED:** This content uses AI for structure and refinement, but still requires additional manual editing.

---

For compute-constrained systems (such as drones), selecting the right hardware is just as critical as the algorithms. The following considerations drove our hardware selection:

*   **Global Shutter:** The camera must have a global shutter. In rapid and quick trajectories (like drone flight), rolling shutter distortion will severely degrade pose calculations.
*   **Hardware Clock Synchronization:** Camera and IMU data must be perfectly in sync. Because they run at different frequencies (e.g., 30fps for the camera vs. 200Hz for the IMU), "in sync" means the timestamps for both data streams must originate from the exact same hardware clock. 
    *   *Latency:* We cannot simply timestamp the data when it arrives at the compute unit. Cameras have internal processing overhead adding milliseconds of latency. We use external trigger pins to sync the clocks of both devices precisely when the frame/data is captured.
*   **Resolution Limits:** A large resolution camera is not necessary and is actually detrimental in compute-constrained systems. Finding and matching features in high-resolution frames requires immense computational power.
*   **Monochrome over RGB:** Native black-and-white (monochrome) cameras are preferred. VIO feature tracking relies on luminance, not color. A monochrome sensor offers a larger light capture area per pixel compared to an RGB sensor of the same size (which divides pixels into color filters). This larger capture area increases dynamic range and reduces sensor noise.