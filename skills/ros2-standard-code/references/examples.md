# Additional implementation examples

These are pattern examples. They do not replace verification against the target-distro API.
Links here point at Rolling. Replace `/rolling/` with the target distro before you read them.
See `source-index.md` for the full catalog.

## 1. Package naming decisions

### Robot model / URDF package

Preferred:

```text
atlas_description
```

Use when the package owns robot-description resources such as URDF/xacro and meshes.

Avoid:

```text
atlas_files
atlas_utils
atlas_model_stuff
```

The preferred name communicates a [REP-144](https://reps.openrobotics.org/rep-0144/)-recognized package role.

### Robot startup package

Preferred:

```text
atlas_bringup
```

Use when the package's primary purpose is launching/configuring the robot as a system.

If a package contains only reusable launch descriptions and is not specifically robot startup, `*_launch` may be more appropriate.

### Simulator integration with ros2_control

A name such as:

```text
example_sim_ros2_control
```

can be reasonable when it follows an established integration-package family.
This is a ros2_control ecosystem convention, not a REP-144 special suffix.
Search peer integrations before finalizing the name.

## 2. Minimal CMake style example

```cmake
cmake_minimum_required(VERSION <target-distro-minimum>)
project(my_robot_control)

find_package(ament_cmake REQUIRED)

add_library(command_limiter SHARED
  src/command_limiter.cpp
)

target_include_directories(command_limiter
  PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
)

install(
  TARGETS command_limiter
  EXPORT export_command_limiter
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY include/
  DESTINATION include/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

Notes:

- Replace the minimum CMake version after checking the target distro ([REP-2000](https://reps.openrobotics.org/rep-2000/) through Kilted).
- Add dependencies as modern imported targets or repository-approved helpers appropriate to the distro.
- Do not copy a Kilted/Rolling CMake idiom into Humble without checking compatibility.
- Add export declarations needed by consumers.
- Handle shared-library symbol visibility when required.

## 3. Manifest style example

```xml
<?xml version="1.0"?>
<package format="3">
  <name>my_robot_control</name>
  <version>0.1.0</version>
  <description>Control utilities for My Robot.</description>

  <maintainer email="maintainer@example.com">Maintainer Name</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <!-- Add direct runtime/build dependencies here. -->

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

Do not add dependencies merely because they are transitively available on the developer's machine.

## 4. ros2_control controller skeleton: API must be verified

The shape of a controller is stable across distros:

```cpp
namespace my_robot_controller
{

class MyRobotController : public controller_interface::ControllerInterface
{
public:
  controller_interface::InterfaceConfiguration
  command_interface_configuration() const override;

  controller_interface::InterfaceConfiguration
  state_interface_configuration() const override;

  controller_interface::CallbackReturn on_init() override;

  controller_interface::return_type update(
    const rclcpp::Time & time,
    const rclcpp::Duration & period) override;
};

}  // namespace my_robot_controller
```

Before turning this into compilable code, open the target distro's
[`ControllerInterface` API](https://control.ros.org/rolling/doc/api/) and a same-distro
[controller example](https://control.ros.org/rolling/doc/ros2_controllers/doc/writing_new_controller.html).
Add lifecycle callbacks only as needed and with the exact target-distro signatures.

## 5. Real-time data handoff pattern

Bad pattern inside synchronous `update()`:

```cpp
auto response = blocking_network_call();
RCLCPP_INFO(get_node()->get_logger(), "response: %s", response.c_str());
std::vector<double> temporary_buffer(joints_.size());
```

Preferred architecture:

```text
non-RT subscription/network callback
        |
        v
realtime-safe handoff / preallocated state
        |
        v
bounded synchronous update()
        |
        v
command interfaces
```

If the computation itself is inherently long or blocking, consider the [ros2_control asynchronous-controller mechanism](https://control.ros.org/rolling/doc/ros2_control/controller_manager/doc/running_controllers_asynchronously.html)
rather than forcing it into the controller manager's synchronous loop.

## 6. Source-selection example

Question: "Should this function be `GetState()` or `get_state()`?"

Decision process:

1. [ROS 2 Code Style](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Code-Style-Language-Versions.html) explicitly allows both CamelCase and snake_case functions.
2. ROS 2 core packages commonly use snake_case.
3. Inspect the target repository.
4. If neighboring APIs use snake_case, choose `get_state()`.
5. If an established package consistently uses CamelCase, preserve that style.
6. Do not call one spelling "the mandatory ROS standard."

Question: "Should I name this package `foo_control`?"

Decision process:

1. REP-144 has no generic `_control` suffix rule.
2. Determine what the package actually contains.
3. If it is a ros2_control controller, investigate `foo_controller`.
4. If it is a robot startup package, investigate `foo_bringup`.
5. If it is a ros2_control integration, compare official `*_ros2_control` peers.
6. Search the target ecosystem before deciding.
