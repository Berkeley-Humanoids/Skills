---
name: robotics-conventions
description: Use for robotics / robot-learning code when handling frames, units, axes, signs, quaternions, URDFs, sim-to-real, RL/IL policies, kinematics, hardware, visualization, or data pipelines.
---

# Robotics conventions

- Prefer explicit names in variables, methods, classes, and configs.
  - Use `_left` / `_right`, not `_l` / `_r`.
  - Avoid ambiguous abbreviations unless they are standard in robotics.

- Use SI units everywhere: meters, seconds, radians, kilograms, Newtons, and Newton-meters.
  - Non-SI values are only allowed at third-party, UI, logging, or hardware API boundaries.
  - Mark non-SI variables with suffixes such as `_deg`, `_ms`, `_mm`.

- Use scalar-first quaternion order: `(qw, qx, qy, qz)`.
  - Normalize quaternions before storing or applying them.
  - Convert explicitly at library/message boundaries when another order is required.

- Use REP-103 robot frames: `x` forward, `y` left, `z` up, right-handed.
  - Blender and Onshape use `y` forward, `z` up.
  - Prefer realigning CAD assets or motion data during export, not patching frame conversions in runtime code.

- Training environment setup should follow this order:
  - `Command`
  - `Observation`
  - `Action`
  - `Metrics`
  - `Rewards`
  - `Terminations`
  - `Events`
  - `Curriculum`

- Robot joint ordering should follow a depth-first body traversal.
  - Example: left arm shoulder-to-hand, right arm shoulder-to-hand, left leg hip-to-foot, right leg hip-to-foot.
  - Keep the ordering defined in one source of truth and reference it everywhere.

- Use and reference ROS 2 documentation, not ROS 1 documentation.

- Prefer environment managers:
  - `uv` for Python environments.
  - `pixi` for C/C++ and ROS 2 environments.

- Preferred data formats:
  - `.mcap` for raw data recording, especially when interfacing with C/C++ code.
  - `LeRobotDataset` / `.parquet` for training datasets stored as Hugging Face dataset repositories.
  - `.rbl` for saved Rerun visualizations that combine `.mcap` data with a Rerun layout.

- Preferred visualization tools:
  - Rerun for multimodal data visualization, especially require custom Blueprint panels.
  - Viser for only motion and pose visualization.
  - MuJoCo for physics-based interactive visualization.
