# Advanced Teleoperation Control for Redundant Manipulators (Kinova Gen3)

## 📌 Project Overview
This project focuses on developing an advanced teleoperation algorithm for robotic manipulators (specifically 6-DOF and 7-DOF Kinova Gen3). The primary goal is to reduce the operator's cognitive load and improve precision in complex, remote manipulation tasks. 

The system implements Closed-Loop Inverse Kinematics (**CLIK**), dynamic on-the-fly **Reference Frame Switching**, and mathematical singularity avoidance using the **Damped Least Squares (DLS)** method.

## 📖 Theoretical Background & Motivation

### 1. The Advantage of Eye-in-Hand Configuration in Precision Teleoperation
In tasks requiring high precision and direct environmental contact (e.g., peg-in-hole, panel operation, grasping irregular objects), teleoperation achieves the best results using an **Eye-in-Hand** camera configuration (camera mounted directly on the end-effector). This setup completely avoids the occlusion problem (the robot arm blocking the view) and allows for highly intuitive control and environmental observation from various angles.

### 2. The Problem of Camera Frame Misalignment
Despite the advantages of an arm-mounted camera, directly implementing an Eye-in-Hand view while leaving the control frame (joystick) mapped to the **Robot Base Frame** leads to drastic performance degradation. 

As explicitly proven by the authors in the article *"An Experimental Study on the Effects of Control Frame and View in Robot Teleoperation"*, visual-motor misalignment causes immense disorientation. The operator must perform mental rotation matrix transformations in real-time, leading to frequent collisions. Recent studies also confirm this: failing to correct the reference frame for an Eye-in-Hand camera increases task completion time, increases unsteadiness by almost 50%, and raises the operator's mental stress by over 60%.

### 3. Solution: Dynamic Reference Frame Switching
To resolve this issue, the project implements an algorithm for dynamic reference frame transformation from the rigid Robot Frame to the View/Tool Frame. This approach aligns with the latest **Shared Control** paradigms. Inspired by architectures presented in top-tier conferences like IEEE ICRA, the system divides the workload: the human decides on vectors and direction in an intuitive visual space, and the algorithm calculates the required transformations (Twists) into the robot's base frame on the fly, ensuring safe and smooth movement.

---

## ⚙️ System Architecture

The controller architecture is based on two modes that the operator can switch on the fly:

* **Base Frame Mode:** Velocity commands from the joypad are interpreted relative to the robot's global base. This is optimal for macro-movements (e.g., rapid approach of the arm to the workspace based on an external mast-camera view).
* **Tool Frame / View Mode:** Joypad commands are mathematically transformed (using the $R_{tool}^{base}$ rotation matrix) to the Eye-in-Hand camera space. A forward push on the joypad always means moving "deeper into the screen," regardless of the arm's current orientation. This is crucial for precise contact tasks.

### Mathematical Safety Layer
1. **CLIK (Closed-Loop Inverse Kinematics):** To eliminate numerical drift resulting from open-loop velocity integration, the system features a feedback loop controlling the position error in the Cartesian space.
2. **DLS (Damped Least Squares):** To protect the hardware from infinite joint velocity spikes near singularities (e.g., full arm extension), the classical Jacobian inversion ($J^{-1}$) is replaced with a robust pseudo-inverse using the Levenberg-Marquardt algorithm.

---

## 🎛️ Inputs and Outputs

**System Inputs:**
* Desired Twist vector (linear and angular velocities) from the joypad: $V_{pad} = [v_x, v_y, v_z, \omega_x, \omega_y, \omega_z]^T$
* Discrete mode switch signal (Button trigger: Base Frame / Tool Frame)
* Robot state (joint positions $q$) to update the Jacobian matrix and forward kinematics in real-time.

**System Outputs:**
* Calculated safe joint angular velocity commands ($\dot{q}$), sent directly to the robot's `joint_group_velocity_controller`.

---

## 🚀 Dependencies & Setup

* **OS:** Ubuntu 20.04 / 22.04
* **Middleware:** ROS Noetic / ROS 2 (Humble)
* **Robot API:** Kortex API (for Kinova Gen3)
* **Math Libraries:** `numpy`, `MoveIt!` 

---
