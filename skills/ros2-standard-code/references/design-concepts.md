# ROS 2 design-article concepts for implementation decisions

This reference expands the design-rationale rules summarized in `../SKILL.md`.

**Authority rule:** these articles explain architectural intent and several designs that became ROS 2 behavior, but some are old proposals.
Use the target distro's documentation and public API for exact syntax, signatures,
supported features, and implementation details.

## 1. Introduction to Real-time Systems

Source: https://design.ros2.org/articles/realtime_background.html

Key concepts:

- Real-time correctness means correct output **and** completion within the required deadline.
- Low latency alone is not a real-time guarantee. Predictability and bounded worst-case latency matter.
- Hard, soft, and firm real-time differ mainly in the consequence of a missed deadline.
- A real-time OS and deterministic application code are both required for true system-level guarantees.
- A common architecture separates non-RT initialization, the RT execution path, and non-RT teardown.
- Typical hazards on the real-time path are page faults, general-purpose heap allocation and release, device and filesystem I/O, thread and process creation, and indefinite blocking.
- Preallocation, memory locking, and prefaulting are platform techniques. They are not universal ROS coding requirements.
- Synchronization can introduce priority inversion. Lock-free or priority-aware mechanisms may help, but verify them for the target platform.
- Measure scheduling jitter and instrument the actual RT loop. Page-fault/resource counters help explain latency spikes.

Operational consequence:
For ros2_control and other control loops, review every operation on the periodic path.
Each one needs a bounded worst-case cost. "Usually fast" is not enough.

## 2. Proposal for Implementation of Real-time Systems in ROS 2

Source: https://design.ros2.org/articles/realtime_proposal.html

Key concepts:

- Define the requirement as both timing (period/deadline + predictability) and failure mode.
- Separate RT and non-RT components so an RT thread is not blocked or preempted by non-RT work.
- Memory allocation and synchronization abstractions are central sources of nondeterminism.
- Atomics are not automatically lock-free. If the guarantee matters, verify the behavior on the target architecture.
- The middleware and the QoS settings both affect determinism. A real-time application cannot ignore the communication stack.
- Lifecycle setup phases are useful because initialization has relaxed timing. Do the allocation, resource setup, and scheduling configuration there.
- Use a test-driven performance process. Stress the system, measure minimum, maximum, and average latency, count missed deadlines, and profile the bottlenecks.
- The document's exact 2016 allocator/executor implementation proposals are historical and must not be copied as current API.

Operational consequence:
When reviewing "real-time safe" code, require evidence or a credible bound for the critical path and prefer current ros2_control/realtime_tools mechanisms over bespoke synchronization.

## 3. Clock and Time

Source: https://design.ros2.org/articles/clock_and_time.html

Key concepts:

- "Real time" as wall-time progression is distinct from "real-time" deterministic computing.
- ROS time lets one algorithm run under wall time, under simulation, and during bag playback.
- When ROS time is off, `ROSTime` follows system time. When simulation time is on, `ROSTime` follows the ROS time source.
- A monotonic/steady clock is appropriate for hardware timeouts and elapsed-time measurements that must not move backward.
- A driver should keep steady-clock timing internal. Where it can, it should publish ROS-network timestamps in ROS time.
- Simulated ROS time may pause, run faster or slower than wall time, and jump forward or backward.
- An algorithm that is sensitive to time discontinuities must react to them. Current ROS 2 exposes clock jump callbacks for this.
- Before a ROS time source has supplied a value, ROS time may be uninitialized/zero.

Operational consequence:
Do not implement a driver timeout with ROS time if pausing simulation must not pause the hardware timeout. Conversely, do not stamp ROS messages with a private monotonic epoch.

Current API check:
- https://docs.ros.org/en/rolling/p/rcl/
- Search the target distro for `Clock`, `TimeSource`, and clock jump callbacks.

## 4. ROS 2 Launch System

Source: https://design.ros2.org/articles/roslaunch.html

Key concepts:

- Launch is an orchestration and event system around processes, ROS nodes, and composable nodes.
- The launch contract includes execution configuration, runtime monitoring, signals, events, and shutdown.
- A process may contain one node or several nodes. In a multi-node executable, launch therefore needs per-node configuration.
- Composition/container loading is part of the launch architecture.
- Launch represents process start, process exit, shutdown, and ROS-specific conditions as events.
- Event handlers are preferable to ad-hoc timing sleeps for sequencing that depends on state.
- Launch descriptions should remain portable: use package-relative lookup/substitutions rather than host-specific paths.

Historical boundary:
Some remote-process/container-service details in this design article were proposals. Verify the target distro's current `launch`, `launch_ros`, and composition APIs.

## 5. ROS 2 Launch XML Format

Source: https://design.ros2.org/articles/roslaunch_xml.html

Key concepts:

- XML is a declarative frontend for the same launch system rather than a separate orchestration model.
- Declarative launch can be easier to read/audit for static configurations.
- Conditions (`if`/`unless`) may control actions.
- `<include>` provides hierarchical reuse and the included description has its own launch-configuration scope.
- Launch arguments are scoped and should be explicitly forwarded to an included description when needed.
- `<group>` can group actions and scope configuration changes.
- Parameters, remaps, environment changes, executables, and ROS nodes are declarative launch entities.
- Built-in/user-defined substitutions provide values evaluated by launch rather than by hard-coded host assumptions.

Current-format check:
Use current `launch_xml` documentation/parser behavior for exact tag/attribute syntax.

## 6. ROS 2 Launch Static Descriptions / Frontend

Source: https://design.ros2.org/articles/roslaunch_frontend.html

Key concepts:

- Different static syntaxes map onto one abstract hierarchy of launch entities and actions.
- A launch frontend needs extensible registration for actions and substitutions.
- Substitutions are part of the frontend model and can be nested/interpolated.
- Treat XML and YAML as two serializations of one frontend, not as two feature sets.

Operational consequence:
Do not build application architecture around XML-vs-YAML parser quirks. If adding a custom launch action/substitution, use the current frontend extension API and make it usable from supported frontends when that is a requirement.

Historical boundary:
Parser registries/macros/decorator sketches in the article are implementation considerations, not guaranteed current API.

## 7. ROS Command Line Arguments

Source: https://design.ros2.org/articles/ros_command_line_arguments.html

Key concepts:

- ROS-specific arguments are explicitly namespaced with `--ros-args` to avoid collisions with application CLI options.
- A trailing `--` separates subsequent non-ROS arguments when needed.
- Standard capabilities include remapping, parameter overrides/files, logging configuration, and other ROS-global/node-specific settings.
- `-r/--remap`, `-p/--param`, and `--params-file` are standard mechanisms.
- In a multi-node executable, rules can be qualified to target the intended node.
- The ROS client libraries parse these arguments. Application code still receives its own non-ROS arguments.

Operational consequence:
Standard ROS arguments already express these needs.
Do not add custom flags such as `--node-name`, `--topic-remap`, or `--param-file`.
Add one only when the application has a genuinely separate contract.

Current guide:
https://docs.ros.org/en/rolling/Developer-Tools/Introspection-and-analysis/Node-arguments.html

## 8. Actions

Source: https://design.ros2.org/articles/actions.html

Current concept guide:
https://docs.ros.org/en/rolling/ROS-Framework/interfaces/About-Actions.html

Key concepts:

- Actions are long-running remote operations with a goal, optional/periodic feedback, a result, and cancellation/preemption capability.
- An action interface contains Goal, Result, and Feedback sections.
- One action server should own an action name. Many clients may use that server.
- The server decides goal acceptance and how concurrent goals are handled.
- Accepted goals move through a state machine: accepted/executing/canceling and succeeded/aborted/canceled terminal outcomes.
- Cancellation is a request, and the server can reject it. After the server accepts a cancellation, it may need to clean up before it finishes.
- An action uses services and topics internally. Normal application code does not touch those transport endpoints.
- Goal/request handling is expected to be quick. Current C++ tutorials explicitly warn that action callbacks should return quickly so they do not starve the executor.
- Feedback rate, feedback QoS, and result retention all affect behavior and resource use. Set them through the target-distro server and client options.

Operational consequence:
An action server must define its concurrency/preemption semantics. "Accept every goal" is incomplete when two goals cannot execute safely at once.

## 9. Intra-process Communications in ROS 2

Source: https://design.ros2.org/articles/intraprocess_communications.html

Current composition guide:
https://docs.ros.org/en/rolling/ROS-Framework/nodes/Working-with-nodes/Composition.html

Key concepts:

- Same-process communication can avoid middleware serialization and copies. This lowers latency and CPU use.
- Intra-process delivery is integrated with publisher/subscription matching and QoS rather than being an unrelated bypass.
- The ownership that a subscriber's buffer and callback require decides whether the middleware moves, shares, or copies a message.
- Unique ownership (`unique_ptr`) can transfer a message to one consumer. Two or more unique-ownership consumers force a copy.
- Shared ownership can avoid copies between compatible consumers. It does not guarantee zero copies in every case.
- Serving both intra-process and inter-process consumers changes the copy/ownership tradeoff.
- `transient_local`/history and other QoS policies still have storage/delivery consequences.
- The article ran its benchmarks on Dashing-era ROS 2. Do not treat those numbers as modern thresholds.

Operational consequence:
Enable `use_intra_process_comms` intentionally for composed components, choose callback/publication ownership with dataflow in mind, then benchmark the actual graph. For modern zero-copy/shared-memory/loaned-message behavior, use target-distro documentation rather than this article.

## Review questions derived from these articles

When reviewing a ROS 2 subsystem, ask:

1. Which operations are timing-critical, and what is the actual deadline/failure policy?
2. Does a periodic/RT path allocate, perform I/O, block, create threads, or take locks with unknown bounds?
3. Are ROS time, steady time, and system time being used for the correct semantics?
4. Can simulation time pause or jump without corrupting the algorithm?
5. Are launch arguments explicit, scoped, portable, and composed through substitutions/includes rather than hard-coded paths?
6. Is ROS CLI configuration using standard ROS argument mechanisms?
7. Should a remote operation be a topic, service, or action?
8. If it is an action, are cancellation and concurrent-goal semantics defined?
9. For composed high-bandwidth nodes, what copies/ownership transitions actually occur?
10. Are performance claims measured on the target distro/hardware rather than inherited from an old design document?
