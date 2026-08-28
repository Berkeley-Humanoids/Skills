---
name: ros2-standard-code
description: Explicit-use-only reference for designing, implementing, and reviewing ROS 2 and ros2_control code against official standards and established community conventions.
---

# ROS 2 Standard Coding

## Goal

Produce ROS 2 code that follows current ROS standards and ROS conventions.
In this skill, **target distro** means the ROS distribution that the code targets.

For each nontrivial decision:

1. Follow the exact target-distro API and the applicable specifications.
2. Follow applicable REPs and official project policy.
3. Match repository policy where stronger rules permit local choice.
4. Use official design documents to understand architectural intent.
5. Use related official packages when no stronger source decides the choice.
6. Search official sources when uncertain.
7. State when a choice is convention, not a standard.

The code must compile. It must also look conventional to a ROS developer.

## Resolve conflicts

Use this order:

1. Exact target-distro API and mandatory compatibility rules.
2. Applicable REP requirements and official target-distro guidance.
3. Repository policy and CI where stronger sources allow local choice.
4. Official ROS 2 design rationale.
5. Closely related official implementations.
6. Broad community convention.
7. Generic C++, Python, or CMake advice.

If official sources disagree, first check whether they target different distros.
If they target the same distro, prefer the newer and more specific source.
State the conflict when it changes the recommendation.

## Research before design

Run this workflow before version-sensitive or public design changes:

1. Determine the target distro.
2. Inspect the repository policy and neighboring code.
3. Read the applicable REP or specification.
4. Read the target-distro ROS 2 documentation.
5. Read the target-distro ros2_control documentation when it applies.
6. Compare two related official implementations when the question is convention.
7. Choose the least-surprising standard solution.
8. Build, test, lint, and load plugins when applicable.
9. Record deliberate deviations from stronger guidance.

Infer the target distro from branches, CI, images, manifests, or dependency versions.
Do not silently assume Rolling when exact APIs matter.

Inspect these repository files when present:

- `CONTRIBUTING.md`
- `README.md`
- `package.xml`
- `CMakeLists.txt`
- format and lint configuration
- two or three related source files or packages

Search official sources in this order:

1. `reps.openrobotics.org`
2. `docs.ros.org/en/<target-distro>/`
3. `control.ros.org/<target-distro>/`
4. `design.ros2.org` for architecture and semantics
5. Official `ros2`, `ament`, `ros-controls`, and `openrobotics/reps` repositories
6. REP-2005 and `index.ros.org` for implementation precedents
7. Maintainer issues or discussions
8. General community sources as supporting evidence only

Do not use one blog, Q&A answer, or downstream repository as the sole authority.

## Select implementation precedents

When no specification defines implementation style, use [REP-2005](https://reps.openrobotics.org/rep-2005/) to find strong reference packages.
REP-2005 is Informational and Active.
A package on that list is not normative in every local choice it makes.

Prefer precedents in this order:

1. The same subsystem in the target distro.
2. The relevant ros2_control package for ros2_control work.
3. A relevant REP-2005 package.
4. Another official ROS 2 package.
5. A mature community package that shows active maintenance.

Useful precedent families include:

- `rclcpp` and `examples_rclcpp_*` for core C++ patterns
- `common_interfaces` and `rcl_interfaces` for interface semantics
- `pluginlib` and `class_loader` for plugin patterns
- `ament_cmake` and `ament_lint` for build and lint patterns
- `ros-controls/ros2_control` and `ros-controls/ros2_controllers` for ros2_control
- maintained vendor and community stacks for repository-scale layout questions:
  `UniversalRobots/Universal_Robots_ROS2_Driver`,
  `ROBOTIS-GIT/dynamixel_hardware_interface`, and the legged stacks under `qiayuanl`

Use a non-official codebase for layout and decomposition questions.
Do not use one for API or style rules.
See `references/source-index.md` for what each codebase does and does not support.

## Core lookup

### Package names and layout

| Decision | Default | Source |
|---|---|---|
| Package name | Lowercase `a-z0-9_`, start with a letter, use no consecutive `_`, and use at least two characters | REP-144, mandatory |
| Package semantics | Use a specific name. Avoid catch-all names such as `utils` and redundant `ros` | REP-144, advised |
| Multi-package repository | Put each package in a same-named directory | ROS 2 docs |
| Single-package repository | The package can be at repository root | ROS 2 docs |
| Public C++ headers | `include/<package_name>/...` | ROS 2 docs |
| C++ implementation | `src/*.cpp` | Convention |
| Tests | `test/`, commonly `test_*.cpp` or `test_*.py` | Convention |
| Launch files | `launch/`, with Python files commonly named `*.launch.py` | Convention |
| Runtime configuration | `config/*.yaml` when the package owns the configuration | Convention |
| Robot description | `<robot>_description` | REP-144, advised |
| Robot bringup | `<robot>_bringup` | REP-144, advised |
| Manifest | `package.xml`. Format 3 is the current ROS 2 norm | [REP-149](https://reps.openrobotics.org/rep-0149/) |

REP-144 marks only three rules as mandatory: the character set, the ban on consecutive `_`, and the two-character minimum.
It states every other naming rule as advice.
Treat the mandatory rules as hard requirements and the rest as strong defaults.

Create only the directories that the package needs.
Do not add empty template directories.

### Package suffixes

[REP-144](https://reps.openrobotics.org/rep-0144/) defines these suffixes as advice, not as mandatory rules:

| Pattern | Meaning |
|---|---|
| `*_driver` | Driver package |
| `*_msgs` | Package that contains messages, services, or actions |
| `*_<library>_plugins` | Plugins for a library |
| `<robot>_robot` | Robot metapackage |
| `<robot>_description` | Robot description, URDF, and meshes |
| `*_tests` | Test-only package |
| `*_launch` | Launch-only package |
| `*_bringup` | Launch-only package that starts a robot |
| `*_tutorials` | Tutorial-only package |
| `*_demos` | Demo-only package |
| `*_ros` | REP-144 upstream-library ROS integration case |

No REP defines these suffixes. Confirm each one against peer packages before you use it:

| Pattern | Meaning |
|---|---|
| `*_interfaces` | Common ROS 2 interface-package name |
| `*_vendor` | Package that vendors an upstream dependency |
| `*_controller` | Package centered on a ros2_control controller |
| `*_ros2_control` | ros2_control integration package |
| `*_control` | No general REP-defined meaning |
| `*_ament` | No general package-suffix rule |

For a non-REP suffix, search related official packages before you introduce it.
Do not infer semantics from a suffix alone.
Do not rename a stable public package only for naming aesthetics.

### Interfaces

| Decision | Default | Source |
|---|---|---|
| Messages | Put `.msg` files in `msg/` | Interface definition rules |
| Services | Put `.srv` files in `srv/` | Interface definition rules |
| Actions | Put `.action` files in `action/` | Interface definition rules |
| Fields | Lowercase alphanumeric with `_`, starting with a letter, with no trailing or consecutive `_` | Interface definition rules |
| Constants | Use the required uppercase interface convention | Interface definition rules |
| Reusable interfaces | Prefer a dedicated interface package | ROS 2 docs |
| Generated C++ headers | Use generated snake-case names under `msg/`, `srv/`, or `action/` | `rosidl` generation |

Treat public interfaces as API.
Check standard interfaces and semantic REPs before you invent fields or meanings.

ROS interfaces order quaternion components as `x`, `y`, `z`, `w`, which puts the scalar last.
`geometry_msgs/Quaternion` defines this order.
Robotics code outside ROS often uses scalar-first order instead.
Convert explicitly at the message boundary. Do not assume that the two orders match.

### C++ style

ROS 2 uses the [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) with the deviations documented in
[ROS 2 Code Style and Language Versions](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Code-Style-Language-Versions.html).

| Decision | Default | Source |
|---|---|---|
| Header extension | `.hpp` | Style guide |
| Source extension | `.cpp` | Style guide |
| Line length | 100 characters | Style guide |
| Indentation | Two spaces and no tabs | Style guide |
| Classes | `CamelCase` | Style guide |
| Functions | Match related code. Prefer `snake_case` when no stronger precedent exists | Style guide, then local code |
| Globals | Lower snake case with `g_` prefix | Style guide |
| Data members | Follow the local ROS package. Trailing `_` is common | Style guide, then local code |
| Access | Prefer private members | Style guide |
| Control statements | Always use braces | Style guide |
| `if`/`else`/`while`/`for` brace | Cuddle the opening brace on the condition line | Style guide |
| Wrapped condition | If the condition wraps across lines, do not cuddle. Put the brace on its own line | Style guide |
| Function/class/enum/struct brace | Put the opening brace on its own line | Style guide |
| Comments | Use `///` or `/** */` for API docs. Use `//` for implementation comments | Style guide |
| Exceptions | Allowed. Do not throw from destructors | Style guide |
| Pointer declarations | Follow ROS spacing, for example `char * c` | Style guide |
| Boost | Avoid unless required | Style guide |
| Warnings | Use at least `-Wall -Wextra -Wpedantic` where supported | Style guide |

ROS 2 contains historical naming differences.
Match related code when official style permits more than one form.
Do not create style-only churn.

Match local header-guard style.
A common pattern is `MY_PACKAGE__MY_HEADER_HPP_`.

### Python and CMake

| Decision | Default | Source |
|---|---|---|
| Python style | [PEP 8](https://peps.python.org/pep-0008/) with ROS 2 changes | Style guide |
| Python line length | 100 characters | Style guide |
| Python quotes | Prefer single quotes when no escaping is needed | Style guide |
| Python imports | One import per line in the documented ROS style | Style guide |
| Python continuations | Prefer hanging indents | Style guide |
| CMake commands | Lowercase, with no space before `(` | Style guide |
| CMake identifiers | `snake_case` | Style guide |
| CMake indentation | Two spaces and no tabs or alignment padding | Style guide |
| `else()` / `endif()` | Do not repeat conditions | Style guide |
| Functions vs macros | Prefer functions | Style guide |
| Minimum CMake version | Read REP-2000 for distros through Kilted. For later distros, read the release docs | [REP-2000](https://reps.openrobotics.org/rep-2000/) |
| Dependency declarations | Declare every direct dependency | REP-149 and developer guide |
| Lint tests | Prefer `ament_lint_auto` with `ament_lint_common` | `ament_cmake` docs |
| Target linkage | Prefer current imported targets. Verify older distros | `ament_cmake` docs, per distro |
| Installed include path | Follow target-distro ament guidance | `ament_cmake` docs, per distro |
| Windows libraries | Handle symbol visibility correctly | ROS 2 docs |

For `ament_cmake` packages:

- Make `project()` match the package name in `package.xml`.
- Call `ament_package()` exactly once.
- Put `ament_package()` last unless documented behavior requires otherwise.
- Install ROS executables to `lib/${PROJECT_NAME}`.
- Put public headers in a package-named include directory.
- Keep private headers out of the installed public include tree.

Do not copy obsolete CMake patterns from an older ROS tutorial.
A clean build must not depend on undeclared packages installed on the developer machine.

### Public API and ABI

Treat these surfaces as external API when users or other packages consume them:

- installed headers and symbols
- messages, services, and actions
- parameters
- plugin names
- node, topic, service, and action names
- executable arguments
- public configuration keys

Keep public API small.
Document intended extension points.
Avoid unnecessary implementation types in public headers.
Consider ABI for existing compiled libraries.
Use deprecation and migration paths for breaking changes.
Follow [SemVer](https://semver.org/) and target-distro compatibility policy.

### Semantic conventions

Use semantic REPs before local conventions:

- [REP-103](https://reps.openrobotics.org/rep-0103/) for SI units and coordinate conventions
- [REP-105](https://reps.openrobotics.org/rep-0105/) for frames such as `base_link`, `odom`, `map`, and `earth`
- [REP-120](https://reps.openrobotics.org/rep-0120/) for humanoid robot frames

Search for subsystem-specific REPs before you define units, frames, or interface semantics.

## Runtime architecture

Use `references/design-concepts.md` for detailed design rationale.
Apply these rules in code and review.

### Real-time

- Define the deadline, allowed jitter, and missed-deadline behavior.
- Do not use low average latency as proof of real-time behavior.
- Separate setup, real-time execution, and teardown.
- Keep unbounded allocation, I/O, waits, and lock contention off the real-time path.
- Verify lock-free behavior before a real-time guarantee depends on an atomic operation.
- Account for priority inversion when real-time code uses locks.
- Measure worst-case latency, jitter, and overruns under stress.
- Treat old allocator and executor proposals as historical.
- Verify current executor, middleware, and ros2_control APIs.

### Time

- Use ROS time for ROS-visible timestamps and simulation-aware algorithms.
- Use steady time for hardware timeouts and monotonic elapsed durations.
- Do not mix clock domains without an explicit conversion.
- Handle ROS-time pause and forward or backward jumps when state depends on elapsed time.
- Reset invalid state after relevant time jumps.
- Do not treat uninitialized simulated time as normal elapsed time.

### Launch

- Treat launch as system orchestration. It does more than start processes.
- Prefer the repository's Python, XML, or YAML convention.
- Use Python when declarative frontends cannot express the required behavior clearly.
- Declare public launch arguments explicitly.
- Keep launch values as substitutions until the launch context evaluates them.
- Pass required arguments explicitly into included launch descriptions.
- Use scoped groups for local namespace, configuration, or environment changes.
- Prefer package-relative substitutions over host-specific absolute paths.
- Use events or conditions for state-dependent sequencing.
- Do not replace state-dependent sequencing with arbitrary sleeps.
- Verify current frontend APIs before you extend launch syntax.

### ROS command-line arguments

- Put ROS arguments inside the `--ros-args ... [--]` scope.
- Use `-r` or `--remap` for remapping.
- Use `-p` or `--param` for parameter overrides.
- Use `--params-file` for parameter files.
- Use standard ROS logging arguments.
- Keep application-specific arguments outside the ROS argument scope.
- Use node-qualified rules when one executable contains multiple nodes.
- Verify current target-distro syntax before you generate exact commands.

### Actions

- Use an action for a long-running operation that needs feedback or cancellation.
- Use a service for a short request-response operation.
- Define Goal, Result, and Feedback by application semantics.
- Make concurrent-goal and preemption policy explicit.
- Return quickly from goal, cancel, and accepted-goal callbacks.
- Move long-running work out of those callbacks.
- Treat cancellation as a cooperative request.
- Preserve `SUCCEEDED`, `ABORTED`, and `CANCELED` meanings.
- Publish feedback at a bounded, useful rate.
- Use the action API. Do not call the underlying service and topic endpoints directly.
- Verify current `rclcpp_action` or `rclpy.action` signatures.

### Intra-process communication

- Use intra-process communication as a same-process optimization.
- Enable it deliberately for composed nodes.
- Keep normal interface and QoS semantics.
- Choose ownership forms with copy behavior in mind.
- Do not assume `shared_ptr` is always faster.
- Do not assume `unique_ptr` always produces zero copies.
- Account for both intra-process and inter-process subscribers.
- Benchmark the actual graph and target distro.
- If you need stricter zero-copy behavior, check current loaned-message and shared-memory support.

## ros2_control

### Architecture

| Decision | Default | Source |
|---|---|---|
| Controller | Derive from the target-distro `controller_interface` base | ros2_control docs |
| Controller loading | Export a `pluginlib` plugin | ros2_control docs |
| Hardware | Implement the correct Actuator, Sensor, or System interface | ros2_control docs |
| Hardware loading | Export a plugin and declare it in ros2_control URDF | ros2_control docs |
| Source layout | `include/<package>/<name>.hpp` and `src/<name>.cpp` | ros2_control docs |
| Lifecycle | Use framework lifecycle callbacks | ros2_control docs |
| Main controller work | Use `update()` | ros2_control docs |
| Hardware I/O | Use framework `read()` and `write()` callbacks | ros2_control docs |
| Interface ownership | Use controller-manager and resource-manager claiming | ros2_control docs |
| Long controller work | Consider asynchronous-controller support | ros2_control docs |
| Parameters | `generate_parameter_library` is a strong current precedent | Convention |
| Real-time handoff | Prefer current `realtime_tools` patterns | `realtime_tools`, then convention |

For synchronous controller-manager paths:

- Keep `update()`, `read()`, and `write()` bounded.
- Do not sleep or perform blocking I/O in the hot path.
- Avoid unnecessary allocation and ownership churn in the hot path.
- Parse messages and requests outside the real-time path.
- Transfer only required data through established real-time-safe mechanisms.
- Use asynchronous-controller support when valid work cannot fit the synchronous loop budget.
- Treat lifecycle transitions and interface claiming as framework semantics.

ros2_control APIs change between distros.
These pages answer most ros2_control questions:

- [API reference](https://control.ros.org/rolling/doc/api/) for exact signatures
- [Hardware components](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_components_userdoc.html) for hardware types and lifecycle
- [Writing a hardware component](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html)
- [Writing a controller](https://control.ros.org/rolling/doc/ros2_controllers/doc/writing_new_controller.html)
- [Asynchronous controllers](https://control.ros.org/rolling/doc/ros2_control/controller_manager/doc/running_controllers_asynchronously.html) for long or blocking work
- [Migration notes](https://control.ros.org/rolling/doc/ros2_control/doc/migration.html) for what changed between distros
- [realtime_tools](https://control.ros.org/rolling/doc/realtime_tools/doc/index.html) for the real-time handoff API
- [ros2_control_demos](https://github.com/ros-controls/ros2_control_demos) for complete worked examples

**Warning:** every link above points at Rolling, which is the development distro.
Replace `/rolling/` with the target distro before you read any of them.
Rolling signatures do not always match a released distro.

Before you write a controller or hardware plugin:

1. Open the target-distro API reference.
2. Verify the base class and all overridden signatures.
3. Verify lifecycle return types.
4. Verify state and command interface ownership.
5. Verify plugin export and CMake helpers.
6. Compare a current example in the same distro.
7. When you consult a vendor or community driver, read its target-distro branch.

Do not copy a Humble controller skeleton into Jazzy, Kilted, or Rolling without verification.

## Review checklist

Before you finish a ROS 2 change, check these items:

### Naming and layout

- [ ] Package names follow [REP-144](https://reps.openrobotics.org/rep-0144/).
- [ ] Package suffixes have documented or established meanings.
- [ ] Files match related ROS package layouts.
- [ ] Public headers are under `include/<package>/`.
- [ ] The change does not add empty template directories.

### API and compatibility

- [ ] You know the target distro for every piece of version-sensitive code.
- [ ] Exact framework signatures match the target distro.
- [ ] Public headers and symbols are intentionally public.
- [ ] New API is minimal and documented.
- [ ] The change accounts for ABI impact on existing compiled libraries.
- [ ] The change treats parameters, plugin names, graph names, and CLI behavior as contracts.

### Style and build

- [ ] C++, Python, and CMake match ROS policy and local style.
- [ ] The change avoids unrelated style churn.
- [ ] Comments explain intent or constraints.
- [ ] The manifest declares every direct dependency.
- [ ] Shared libraries export symbols portably where the package needs it.
- [ ] Tests and lint checks cover the change.
- [ ] The package builds in a clean environment when practical.

### ros2_control

- [ ] The controller or hardware abstraction is correct.
- [ ] Plugin export and plugin description are correct.
- [ ] Lifecycle behavior is intentional.
- [ ] Synchronous hot paths stay bounded and never block.
- [ ] The code claims interfaces through the framework APIs.
- [ ] The implementation matches the target-distro API.
- [ ] The design considers async execution when synchronous work exceeds the loop budget.
- [ ] A test loads the plugin when the change adds or changes a plugin.

## Answer design questions

When this skill is active:

1. Give the recommendation first.
2. Say whether it is a requirement or a convention.
3. Name the primary official source or official precedent.
4. Mention alternatives only when the ecosystem uses them.
5. Verify the target distro before version-sensitive recommendations.
6. State weak or conflicting evidence instead of inventing a rule.

Apply routine checks as you write and review code.
Do not write a standards essay unless the user asks for the rationale.

## Keep the skill current

Search current official sources when a decision depends on:

- the target distro
- C++ or CMake minimum versions
- ros2_control signatures
- deprecated APIs
- new or changed REPs
- package suffixes not defined by REP-144

Use `references/source-index.md` for sources.
Use `references/design-concepts.md` for ROS 2 design rationale.
Use `references/examples.md` for implementation examples.
