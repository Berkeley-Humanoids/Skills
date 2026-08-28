# Official source index

This file is the lookup catalog for `ros2-standard-code`.
When a decision is version-sensitive, read the target-distro version of the documentation.

## REP corpus

| Source | What it governs / why it matters | Status/use |
|---|---|---|
| [REP index](https://reps.openrobotics.org/) | Current REP catalog and status metadata | Always check status/applicability |
| [REP-0001:2025](https://reps.openrobotics.org/rep-0001-2025/) | Current REP process | Helps interpret modern REP types/statuses |
| [REP-103](https://reps.openrobotics.org/rep-0103/) | Standard units and coordinate conventions | Active/use for interoperable semantics |
| [REP-105](https://reps.openrobotics.org/rep-0105/) | Standard mobile robot coordinate frames | Active/use when applicable |
| [REP-120](https://reps.openrobotics.org/rep-0120/) | Coordinate frames for humanoid robots | Informational/Active; extends REP-105 |
| [REP-122](https://reps.openrobotics.org/rep-0122/) | Filesystem hierarchy | **Draft**; corroborate with current ROS 2 docs and package precedent |
| [REP-144](https://reps.openrobotics.org/rep-0144/) | ROS package naming | Active; contains mandatory rules plus recommendations |
| [REP-149](https://reps.openrobotics.org/rep-0149/) | Package Manifest Format 3 | Final |
| [REP-2000](https://reps.openrobotics.org/rep-2000/) | ROS 2 release/platform/toolchain policy through Kilted | Active; from Lyrical onward consult release docs |
| [REP-2004](https://reps.openrobotics.org/rep-2004/) | Package quality categories | Active |
| [REP-2005](https://reps.openrobotics.org/rep-2005/) | ROS 2 common packages: the curated list this skill uses to pick precedents | Informational/Active; a curated package list, not a style mandate |

Do not assume all indexed REPs apply to ordinary ROS 2 application code.
Search the index by subject before making decisions about messages, frames, QoS, security, hardware acceleration, bag formats, etc.

## Language and general standards

ROS 2 style is defined as a set of deviations from these documents.
Read the ROS 2 Code Style page together with them.
Where they disagree, ROS 2 wins.

| Source | Use |
|---|---|
| [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) | Base for ROS 2 C++ style |
| [PEP 8](https://peps.python.org/pep-0008/) | Base for ROS 2 Python style |
| [Semantic Versioning](https://semver.org/) | The version contract the Developer Guide requires |
| [CMake documentation](https://cmake.org/cmake/help/latest/) | Upstream CMake reference behind `ament_cmake` |
| [ROS 2 Code Style and Language Versions](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Code-Style-Language-Versions.html) | The ROS deviations from the above, plus language versions |

## ROS 2 project guidance

| Source | Use |
|---|---|
| [ROS 2 Rolling docs index](https://docs.ros.org/en/rolling/index.html) | Top-level documentation map |
| [Contributing](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing.html) | Community tenets; prefer community best practices over ad-hoc processes |
| [Developer Guide](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Developer-Guide.html) | SemVer, API/ABI, docs, tests, repository practices |
| [Code Style and Language Versions](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Code-Style-Language-Versions.html) | C/C++/Python/CMake/document style |
| [Quality Guide](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Quality-Guide.html) | Static/dynamic analysis and quality patterns |
| [Build Farms](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Build-Farms.html) | Clean/minimal release-build expectations |
| [Windows Tips and Tricks](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Windows-Tips-and-Tricks.html) | Symbol visibility and portability |
| [Making a PR](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Contributing-to-code/Making-a-PR.html) | Scope, tests, lint, review workflow |
| [Reviewing a PR](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Contributing-to-code/Reviewing-a-PR.html) | Review criteria |
| [Documenting a ROS 2 package](https://docs.ros.org/en/rolling/Developer-Tools/Package-documentation/Documenting-a-ROS-2-Package.html) | Package-level documentation; if URL moves, search docs by title |
| [ament_cmake user documentation](https://docs.ros.org/en/rolling/Developer-Tools/Build/Ament-CMake-Documentation.html) | Canonical package CMake layout, target linking, install/export, lint/test patterns |
| [Interfaces](https://docs.ros.org/en/rolling/ROS-Framework/interfaces/About-Interfaces.html) | `.msg`/`.srv`/`.action` layout, field and constant naming |
| [Creating an action](https://docs.ros.org/en/rolling/ROS-Framework/interfaces/actions/Working-with-actions/Creating-an-Action.html) | Interface-package separation pattern; if URL moves, search docs by title |
| [ROS Index](https://index.ros.org/) | Find package source, supported distros, metadata |

Rolling reorganized its documentation tree, so a Rolling path often does not exist under a
released distro (and the reverse). Do not rewrite a URL by swapping `/rolling/` for the
target distro without checking it. Search `docs.ros.org` by page title instead.

### Design rationale

| Source | Use |
|---|---|
| [design.ros2.org](https://design.ros2.org/) | Architecture articles behind ROS 2 behavior |

Read `design-concepts.md` in this directory first: it summarizes the relevant articles and
marks which parts are historical proposals rather than current API.

### Source repositories

When rendered docs are unavailable or ambiguous, inspect the official source:

- https://github.com/ros2/ros2_documentation
- https://github.com/openrobotics/reps
- https://github.com/ros2
- https://github.com/ament

Use the branch that matches the target distro whenever possible.

## ros2_control

| Source | Use |
|---|---|
| [ros2_control docs](https://control.ros.org/) | Select the target distro first |
| [Rolling Getting Started](https://control.ros.org/rolling/doc/getting_started/getting_started.html) | Architecture and main abstractions |
| [Rolling core docs](https://control.ros.org/rolling/doc/ros2_control/doc/index.html) | Core concepts and best practices |
| [Hardware Components](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_components_userdoc.html) | Hardware types/lifecycle |
| [Writing a Hardware Component](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html) | Current hardware package/plugin pattern |
| [Asynchronous Controllers](https://control.ros.org/rolling/doc/ros2_control/controller_manager/doc/running_controllers_asynchronously.html) | Long/blocking controller execution |
| [Rolling API docs](https://control.ros.org/rolling/doc/api/) | Exact current signatures |
| [ros2_control source](https://github.com/ros-controls/ros2_control) | Core implementation precedent |
| [ros2_controllers source](https://github.com/ros-controls/ros2_controllers) | Controller implementation precedent |
| [ros2_control_demos](https://github.com/ros-controls/ros2_control_demos) | Canonical integration examples |
| [realtime_tools](https://github.com/ros-controls/realtime_tools) | Real-time/non-real-time handoff utilities |
| [realtime_tools docs](https://control.ros.org/rolling/doc/realtime_tools/doc/index.html) | Current RT handoff API rather than an older pattern |
| [Writing a new controller](https://control.ros.org/rolling/doc/ros2_controllers/doc/writing_new_controller.html) | Current controller package/plugin pattern |
| [Migration notes](https://control.ros.org/rolling/doc/ros2_control/doc/migration.html) | What changed between distros before porting a controller or hardware component |
| [generate_parameter_library](https://github.com/PickNikRobotics/generate_parameter_library) | The parameter-generation precedent named in the skill |
| [control_toolbox](https://github.com/ros-controls/control_toolbox) | Shared control primitives (PID, filters) before writing your own |

Important: Rolling is a development version.
For released projects, replace `/rolling/` with the target distro and verify the corresponding source branch/tag.

## Core packages and APIs

Every one of these is named somewhere in `../SKILL.md`. Use the API docs for signatures and
the repository for implementation and style precedent. Swap `/rolling/` for the target distro.

| Package | API docs | Source |
|---|---|---|
| `rclcpp` | [API](https://docs.ros.org/en/rolling/p/rclcpp/) | [ros2/rclcpp](https://github.com/ros2/rclcpp) |
| `rclcpp_action` | [API](https://docs.ros.org/en/rolling/p/rclcpp_action/) | [ros2/rclcpp](https://github.com/ros2/rclcpp) |
| `rclpy` (incl. `rclpy.action`) | [API](https://docs.ros.org/en/rolling/p/rclpy/) | [ros2/rclpy](https://github.com/ros2/rclpy) |
| `examples_rclcpp_*`, `examples_rclpy_*` | — | [ros2/examples](https://github.com/ros2/examples) |
| `common_interfaces` | — | [ros2/common_interfaces](https://github.com/ros2/common_interfaces) |
| `rcl_interfaces` | — | [ros2/rcl_interfaces](https://github.com/ros2/rcl_interfaces) |
| `rosidl` (interface generation) | [generated C++](https://docs.ros.org/en/rolling/p/rosidl_generator_cpp/) | [ros2/rosidl](https://github.com/ros2/rosidl) |
| `pluginlib` | [API](https://docs.ros.org/en/rolling/p/pluginlib/) | [ros/pluginlib](https://github.com/ros/pluginlib) |
| `class_loader` | — | [ros/class_loader](https://github.com/ros/class_loader) |
| `ament_cmake` | [user docs](https://docs.ros.org/en/rolling/Developer-Tools/Build/Ament-CMake-Documentation.html) | [ament/ament_cmake](https://github.com/ament/ament_cmake) |
| `ament_lint`, `ament_lint_auto`, `ament_lint_common` | [linters for clean code](https://docs.ros.org/en/rolling/Developer-Tools/Build/Ament-Lint-For-Clean-Code.html) | [ament/ament_lint](https://github.com/ament/ament_lint) |
| `launch`, `launch_xml`, `launch_yaml` | [API](https://docs.ros.org/en/rolling/p/launch/) | [ros2/launch](https://github.com/ros2/launch) |
| `launch_ros` | [about launch](https://docs.ros.org/en/rolling/Developer-Tools/About-Launch.html) | [ros2/launch_ros](https://github.com/ros2/launch_ros) |

## Useful implementation precedents

Use closely related packages rather than a generic "popular ROS package":

- [`rclcpp`](https://github.com/ros2/rclcpp): core C++ API and build/style precedent.
- [`pluginlib`](https://github.com/ros/pluginlib): plugin export/loading.
- [`class_loader`](https://github.com/ros/class_loader): dynamic loading internals.
- [`ros2_controllers/forward_command_controller`](https://github.com/ros-controls/ros2_controllers/tree/master/forward_command_controller): relatively small controller example.
- [`ros2_controllers/joint_trajectory_controller`](https://github.com/ros-controls/ros2_controllers/tree/master/joint_trajectory_controller): larger parameterized controller.
- [`ros2_control/hardware_interface`](https://github.com/ros-controls/ros2_control/tree/master/hardware_interface): hardware base APIs.
- [`ros2_control_demos`](https://github.com/ros-controls/ros2_control_demos): complete hardware/controller examples.

A precedent is evidence of convention, not a substitute for a specification.

## Reference codebases outside ros-controls

These are real ros2_control and ROS 2 integrations.
They answer questions that no specification answers: how to split packages, where to draw
the driver and controller boundary, how to branch per distro, and how to lay out launch
and configuration files. None of them is normative.
Weigh each by how well it is maintained and how close it is to your target distro.

### UniversalRobots — https://github.com/UniversalRobots

The strongest vendor-driver precedent in this list. BSD-3-Clause, actively maintained,
released into the ROS 2 buildfarm, and branched per distro.

| Repository | Use as precedent for |
|---|---|
| [Universal_Robots_ROS2_Driver](https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver) | Multi-package driver repository; a `hardware_interface` System plugin with a vendor client library behind it |
| [Universal_Robots_ROS2_Description](https://github.com/UniversalRobots/Universal_Robots_ROS2_Description) | Keeping `*_description` (URDF, xacro, meshes, ros2_control macro) separate from the driver |
| [Universal_Robots_Client_Library](https://github.com/UniversalRobots/Universal_Robots_Client_Library) | Keeping the protocol/transport library ROS-independent and testable on its own |
| [Universal_Robots_ROS2_GZ_Simulation](https://github.com/UniversalRobots/Universal_Robots_ROS2_GZ_Simulation) | Simulation integration as a separate repository reusing the same description |
| [Universal_Robots_ROS2_Tutorials](https://github.com/UniversalRobots/Universal_Robots_ROS2_Tutorials) | Tutorial packages kept out of the driver repository |

Specific patterns worth copying:

- Package split inside one repository: `ur` (metapackage), `ur_robot_driver`,
  `ur_controllers`, `ur_calibration`, `ur_dashboard_msgs`, `ur_moveit_config`.
  Each name states its role and matches REP-144 families.
- The `ur` metapackage declares only `exec_depend` on its members.
- Plugin descriptions are named for their role and exported per package:
  `hardware_interface_plugin.xml` in `ur_robot_driver`,
  `controller_plugins.xml` in `ur_controllers`.
- One branch per distro (`humble`, `jazzy`, `kilted`, `lyrical`, `main`) plus
  per-distro `.repos` files. Read the branch that matches your target distro, not `main`.

### ROBOTIS-GIT — https://github.com/ROBOTIS-GIT

Actuator-level ros2_control integration and a widely used reference platform.

| Repository | Use as precedent for |
|---|---|
| [dynamixel_hardware_interface](https://github.com/ROBOTIS-GIT/dynamixel_hardware_interface) | A single-package `hardware_interface` System plugin over a vendor SDK, with `humble`, `jazzy`, and `main` branches |
| [dynamixel_interfaces](https://github.com/ROBOTIS-GIT/dynamixel_interfaces) | A minimal companion `*_interfaces` package for one hardware package |
| [dynamixel_hardware_interface_demos](https://github.com/ROBOTIS-GIT/dynamixel_hardware_interface_demos) | Demo packages kept separate from the hardware package |
| [DynamixelSDK](https://github.com/ROBOTIS-GIT/DynamixelSDK) | The non-ROS vendor SDK the hardware interface wraps |
| [turtlebot3](https://github.com/ROBOTIS-GIT/turtlebot3) | A long-lived platform repository split into `*_description`, `*_bringup`, `*_node`, and `*_msgs` |
| [open_manipulator](https://github.com/ROBOTIS-GIT/open_manipulator) | Manipulator bringup, MoveIt config, and ros2_control wiring |
| [cyclo_control](https://github.com/ROBOTIS-GIT/cyclo_control) | Layering a solver-agnostic core, its ROS wrapper, and a `*_vendor` package in one repository |

Notes:

- `dynamixel_hardware_interface` is single-package at repository root, with the plugin
  description (`dynamixel_hardware_interface_plugin.xml`) beside `package.xml`.
  The UR driver uses a multi-package layout instead. Both layouts are conventional.
- `cyclo_control` splits `cyclo_motion_controller_core` (no ROS) from
  `cyclo_motion_controller_ros` and `cyclo_motion_controller_ros_py`, and vendors a
  solver as `osqp_eigen_vendor`. This is a concrete `_vendor` and `_ros` precedent.
- Some ROBOTIS repositories still carry ROS 1 packages, and some are archived.
  Check the `package.xml` format and the branch before you treat one as a ROS 2 precedent.

### qiayuanl — https://github.com/qiayuanl

Research-grade legged-robot control. Useful for control-stack *architecture*, not for
ROS 2 API or style questions. These are individual-maintainer repositories: no ROS
buildfarm release, no distro branches, and local style that departs from ROS 2 policy.

| Repository | Status | Use as precedent for |
|---|---|---|
| [legged_control](https://github.com/qiayuanl/legged_control) | **ROS 1 (catkin, `ros_control`). README states it is no longer supported.** | Decomposition of a legged stack into `legged_hw`, `legged_controllers`, `legged_estimation`, `legged_wbc`, `legged_interface`, `legged_common`, `legged_examples` |
| [legged_perceptive](https://github.com/qiayuanl/legged_perceptive) | ROS 1, extends `legged_control` | Adding a perception layer on top of an existing control stack |
| [legged_template_controller](https://github.com/qiayuanl/legged_template_controller) | ROS 2, `ament_cmake` format 3 | Minimum shape of an out-of-tree `controller_interface::ControllerInterface` package |
| [mujoco_ros2_control](https://github.com/qiayuanl/mujoco_ros2_control) | ROS 2 | A simulator-backed `hardware_interface` plus a separate `*_demos` package |
| [traj_tracking_controller](https://github.com/qiayuanl/traj_tracking_controller) | ROS 2 | A small single-purpose controller package with its own plugin description |

Read `legged_control` to see how it divides an NMPC, WBC, and state-estimation stack
across packages, and where it puts the real-time boundary. Do **not** port its APIs: `ros_control`
`RobotHW` is not ros2_control `hardware_interface`, and the ROS 1 controller lifecycle
differs from the ROS 2 lifecycle. Translate the architecture, then verify every
signature against target-distro ros2_control.

`legged_template_controller` shows a working ros2_control plugin export
(`pluginlib_export_plugin_description_file(controller_interface ...)`), but it is a
template: its `package.xml` metadata is placeholder text, it uses `ament_cmake_auto`
rather than plain `ament_cmake`, and its indentation does not follow ROS 2 style.
Take the structure, not the boilerplate.

### How to weigh these against official sources

1. A specification or REP outranks all of them.
2. `ros-controls/ros2_control`, `ros2_controllers`, and `ros2_control_demos` outrank them
   for framework API and lifecycle questions.
3. Use a vendor repository when the question is "how do real drivers organize this?"
4. Prefer a repository that is buildfarm-released and distro-branched over one that is not.
5. Check the branch matching your target distro before quoting any of them.
6. Confirm a pattern in two independent repositories before calling it a convention.

## Package-suffix evidence notes

### `_vendor`

`_vendor` is not defined by REP-144 as a general suffix.
It is nevertheless an established ROS 2 packaging convention.
For example, ROS 2 Jazzy introduced `gz_*_vendor` packages to make Gazebo dependencies consumable by ROS 2 packages:

- https://docs.ros.org/en/rolling/Get-Started/Releases/Release-Jazzy-Jalisco.html

Other vendor packages exist in the ROS ecosystem.
A downstream example is `osqp_eigen_vendor` inside
[ROBOTIS-GIT/cyclo_control](https://github.com/ROBOTIS-GIT/cyclo_control), which packages a
solver dependency for the surrounding control packages.
Use the suffix for actual dependency-vendoring/packaging roles, not for arbitrary wrappers.

### `_ament`

No general REP rule was found that gives every `*_ament` package a standard semantic role.
Do not claim one.
Search the specific ament repository/tool precedent before introducing such a name.

### `_control`

No general REP rule defines `_control`.
In ros2_control ecosystems, prefer names that state the actual function (`*_controller`, `*_hardware`, `*_description`, `*_bringup`, integration names such as `*_ros2_control`) and verify peer packages.

## Version-sensitive CMake note

Kilted/Rolling documentation deprecates `ament_target_dependencies()` in favor of modern CMake imported targets.
For older distros, check the target distro before changing an established build style.

## Maintenance procedure

Periodically re-check:

1. REP-144 and current REP index status.
2. ROS 2 Code Style and Developer Guide.
3. The release and platform page for the target distro.
4. ros2_control migration/release notes and current API.
5. Deprecation warnings emitted by current builds.
6. 2-3 relevant official packages for conventions that have no normative document.
7. The reference codebases listed above: whether they still track a supported distro, and
   whether any has been archived or superseded.

Update this index when an official source moves or supersedes an older one.
