# IsaacSim_SO101_MoveIt

A ROS 2 Humble workspace that connects the **SO-101** robotic arm to **MoveIt 2** motion planning, with execution running inside **NVIDIA Isaac Sim**. MoveIt plans trajectories and drives the arm in simulation over ROS 2 topics, giving a ready-to-run **sim-to-sim** setup. A pre-built Isaac Sim USD scene is included, so no manual URDF import is required.

## Overview

The workspace exposes MoveIt's `ros2_control` layer to Isaac Sim through the `topic_based_ros2_control` hardware plugin instead of the usual mock hardware. MoveIt publishes joint commands and reads joint states over dedicated topics; Isaac Sim consumes the commands and returns the simulated state. The result is a closed loop where you plan in RViz and watch the arm move in Isaac Sim.

- **Robot:** SO-101 (SO-ARM101), 6 joints
- **Planner:** MoveIt 2 (KDL kinematics)
- **Simulator:** Isaac Sim (USD scene provided)
- **Bridge:** `topic_based_ros2_control/TopicBasedSystem`
- **Middleware:** ROS 2 Humble

## Repository Structure

```
IsaacSim_SO101_MoveIt/
├── so_101_arm.usd                 # Pre-built Isaac Sim scene for the SO-101
├── src/
│   ├── so_arm_description/        # URDF, meshes, robot description
│   └── so_arm_moveit_config/      # MoveIt 2 configuration and launch files
└── LICENSE
```

## Prerequisites

- **Ubuntu 22.04** with **ROS 2 Humble**
- **MoveIt 2** and `ros2_control` / `ros2_controllers`
- **`topic_based_ros2_control`** (bridges MoveIt to Isaac Sim)
- **NVIDIA Isaac Sim** with the ROS 2 bridge enabled

Install the core ROS 2 dependencies:

```bash
sudo apt update
sudo apt install \
  ros-humble-moveit \
  ros-humble-ros2-control \
  ros-humble-ros2-controllers \
  ros-humble-gripper-controllers \
  ros-humble-topic-based-ros2-control
```

## Build

```bash
# Source ROS 2
source /opt/ros/humble/setup.bash

# From the workspace root
rosdep install --from-paths src --ignore-src -r -y
colcon build
source install/setup.bash
```

## Running

### 1. Start Isaac Sim

Open `so_101_arm.usd` in Isaac Sim (with the ROS 2 bridge enabled) and press **Play**. The scene publishes and subscribes to the topics MoveIt expects:

| Direction        | Topic                  |
| ---------------- | ---------------------- |
| State (Sim → MoveIt)   | `/isaac_joint_states`  |
| Command (MoveIt → Sim) | `/isaac_joint_command` |

### 2. Launch MoveIt

In a separate terminal:

```bash
source install/setup.bash
ros2 launch so_arm_moveit_config demo.launch.py
```

This brings up `move_group`, RViz with the MoveIt Motion Planning panel, the controller manager, and the configured controllers. Plan a goal in RViz and execute it — the arm moves in Isaac Sim.

> The `ros2_control` hardware is pre-configured for simulation via the topic bridge. To target real hardware instead, replace the `topic_based_ros2_control` plugin in `so101_new_calib.ros2_control.xacro` with the appropriate hardware interface.

## Configuration Reference

### Planning Groups

| Group     | Joints                                                     | DOF |
| --------- | --------------------------------------------------------- | --- |
| `arm`     | `Rotation`, `Pitch`, `Elbow`, `Wrist_Pitch`, `Wrist_Roll` | 5   |
| `gripper` | `Jaw`                                                     | 1   |

### Named States

| State          | Group     | Description                       |
| -------------- | --------- | --------------------------------- |
| `arm_home`     | `arm`     | All arm joints at 0               |
| `open`         | `gripper` | Jaw at 1.7453 rad                 |
| `close`        | `gripper` | Jaw at −0.1745 rad                |
| `gripper_home` | `gripper` | Jaw at 0                          |

### Controllers

| Controller              | Type                                | Interface                |
| ----------------------- | ----------------------------------- | ------------------------ |
| `arm_controller`        | `JointTrajectoryController`         | `FollowJointTrajectory`  |
| `gripper_controller`    | `GripperActionController`           | `GripperCommand`         |
| `joint_state_broadcaster` | `JointStateBroadcaster`           | —                        |

Controller manager runs at **100 Hz**. Default velocity and acceleration scaling are set to **0.1** in `joint_limits.yaml` for safe, slow motion — raise these (≤ 1.0) for faster execution.

### Kinematics

KDL plugin (`kdl_kinematics_plugin/KDLKinematicsPlugin`), search resolution and timeout both 0.005.

## Notes

- The scene is intended for **simulation** out of the box. Real-hardware use requires swapping the `ros2_control` hardware plugin (see the note above).
- Joint limits are deliberately conservative; tune `config/joint_limits.yaml` and the scaling factors to taste.

## Credits

- **[TheRobotStudio](https://github.com/TheRobotStudio/SO-ARM100)** — original SO-ARM100 / SO-101 hardware and simulation assets
- **[LycheeAI](https://www.youtube.com/@LycheeAI)** — SO-ARM ROS 2 / Isaac Sim integration groundwork
- **[MoveIt 2](https://moveit.ai/)** — motion planning framework

## License

Released under the [MIT License](LICENSE).
