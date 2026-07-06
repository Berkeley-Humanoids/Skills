---
name: pixi-workspace
description: Use when creating, editing, or reviewing pixi.toml manifests, pixi tasks, workspace scripts (activate/doctor/canup), or any Berkeley Humanoids pixi+ROS2 workspaces.
---

# Pixi-ROS2 Workspace Guideline

Every Berkeley Humanoids pixi workspace exposes the same pixi task
interface: a universal maintenance core plus a reserved vocabulary for
the deployment and utility tasks each workspace defines. Tasks are thin
wrappers over canonical commands (`ros2 launch/run`, `colcon`), never a
replacement layer; everything keeps working verbatim inside `pixi shell`.

## General Guideline

- Every task carries `description = "…"`: `pixi task list` is the menu.
  `_`-prefixed tasks are hidden plumbing.
- Launch-wrapping tasks are arg-less command strings so trailing
  `key:=value` forwards verbatim. Don't use typed `args` on them.
- **No deployment or utility task may auto-build** — a hardware bringup
  must never trigger a surprise rebuild. Only `build` chains `setup`.
  Display error message and provide guidance if we lack something.
- Where a new entry point goes (first rule that matches):
  1. operates on the workspace (fetch/build/verify) → maintenance task,
     universal names only;
  2. everyday robot scenario or utility → workspace task (reserved
     vocabulary where it applies, free kebab-case otherwise);
  3. needs root / mutates host state (CAN buses, kernel) →
     `scripts/*.sh` run with explicit `sudo`, never a task;
  4. rarely used / one-shot → no task; document the canonical command
     (`pixi run ros2 …` or `pixi shell` then `ros2 …`).
- Forbidden names: `install`/`shell` (shadow pixi verbs), `run`/`start`
  (redundant next to the vocabulary), `launch-*` prefixes, colon
  namespacing (`build:all`), same-name tasks in multiple environments.

## Maintenance Tasks

The universal core — every developer-facing workspace provides these
five, names **and** semantics reserved:

| Task | Meaning | Rules |
|---|---|---|
| `setup` | fetch/prepare sources beyond pixi itself (`vcs import`, fetch scripts) | idempotent and cached (canonical shape below); safe to re-run; chained by `build` |
| `build` | build the workspace's default libraries with colcon | `depends-on = ["setup"]`; style flags come from `config/colcon-defaults.yaml`, only semantic selection flags (`--packages-skip-regex`, `--packages-up-to`) appear inline |
| `test` | run the workspace's tests, linters excluded | `colcon test … --ctest-args -LE linter`; omit only if the workspace truly has no tests |
| `clean` | wipe the colcon overlay | exactly `rm -rf build install log`; never touches `src/` or `.pixi/` |
| `doctor` | workspace health check (`scripts/doctor.sh`) | checks: workspace root, env active, `/opt/ros` contamination (hard fail), sources imported, overlay built + sourced, `ROS_DOMAIN_ID`; warnings for fixable, non-zero for fatal |

Day one is one command: `pixi run build` (pixi self-heals the env;
`setup` is chained and cached), verified by `pixi run doctor`.

Archetype-local extras are allowed (with descriptions), never required
of other workspaces — dev workspace: `build-all` (adds the
EtherCAT/Prime lane), `test-result` (singular, matches the colcon verb),
`lint` (the `.githooks` ament linter set), `check` (= `test` + `lint`),
`gen-dds`, `test-dds`; buildfarm manifest: `setup` (hidden `_import` +
`_overlay`), `package`, `publish` — the packaging verb is `package`,
never `build*`.

## Robot Deployment Tasks

Reserved names: a workspace that has the concept must use the name; the
stack each name brings up is the workspace's own call (state it in the
task `description`).

| Task | Meaning |
|---|---|
| `sim` / `sim-<scene>` | bring up the robot stack in simulation — environment only, no policy |
| `real` / `real-<scene>` | real hardware bringup (pass `hardware_config:=`, `calibration_file:=`) |
| `policy` / `policy-<task>` | prepare + load the control policy against an already-running stack (pass `wandb_run_path:=` or `checkpoint_file:=`) |
| `deploy-sim` | one-command sim2sim deployment = bringup + policy in one (pass `wandb_run_path:=` or `checkpoint_file:=`) |
| `deploy-real` | one-command real deployment = hardware bringup + policy in one (pass `wandb_run_path:=` / `checkpoint_file:=`, `hardware_config:=`, `calibration_file:=`) |
| `viz` | live state viewer of a running robot (`viewer:=viser` default, optionally `viewer:=rerun`, `viewer:=rviz`) |
| `calibrate` | run calibration procedure; writes `calibration.yaml` |

- Grammar: `<scenario>[-<qualifier>]`, kebab-case, **at most one
  qualifier**, encoding only the primary task variant. All secondary
  parameters stay ROS launch arguments:
  - Good: `pixi run sim-reach robot:=g1 policy:=latest`
  - Bad: `pixi run sim-unitree-g1-reach-latest`
- Unqualified = the workspace's default target.
- `deploy-*` wraps a combined launch file — two blocking bringups can't
  be chained with `depends-on`.

## Debugging and Utility Tasks

Free kebab-case, verb-first where natural; each is a one-line wrapper
over the canonical `ros2 run` command, so its meaning can't drift from
the tool it wraps.

| Task | Meaning |
|---|---|
| `ping-bus` | single-actuator GetDeviceId ping — no Enable, safe on a powered robot |
| `scan-bus` | scan a CAN bus for Robstride device ids (read-only) |
| `profile-bus` | link probe: RTT/jitter (inert) or +lumped delay (PRBS torque) |

Rarely-used companions (the profile report/plot generator, the static
URDF inspector, the MIT slider GUI) deliberately have **no** task —
document their canonical `ros2 run/launch` forms instead.

## Environment

- The canonical `setup` shape (all three parts required — an uncached
  `vcs import` re-fetches every remote and fails offline; without
  `--skip-existing` it resets locally-checked-out branches to pins):

```toml
[tasks.setup]
cmd = "vcs import --skip-existing --input <repo>.repos src && mkdir -p build && touch build/.setup-stamp"
inputs = ["<repo>.repos"]
outputs = ["build/.setup-stamp"]
description = "Import third-party source deps — idempotent, cached on the .repos file"
```

- Overlay sourcing via a guarded activation script (POSIX `setup.sh`,
  not `setup.bash`), never a bare `install/setup.*` entry:

```toml
[activation]
scripts = ["scripts/activate.sh"]
```

```sh
if [ -f "$PIXI_PROJECT_ROOT/install/setup.sh" ]; then
  . "$PIXI_PROJECT_ROOT/install/setup.sh"
fi
```

- Colcon *style* flags (`--symlink-install`, compile-commands) go in
  `config/colcon-defaults.yaml` via `COLCON_DEFAULTS_FILE` in
  `[activation.env]`, so manual `colcon build` == `pixi run build`.
- `[activation.env]` always sets `RCUTILS_COLORIZED_OUTPUT=1` + the
  console format; deployment workspaces pin their robot's
  `ROS_DOMAIN_ID`, the dev workspace leaves it unpinned.
- Channels: `https://prefix.dev/robostack-jazzy` before
  `https://prefix.dev/conda-forge` (binary consumers put
  `https://prefix.dev/berkeley-humanoids` first). Single default
  environment; add features/environments only when dependencies
  genuinely diverge.
- No rosdep, no apt — `pixi.toml` is the dependency source of truth.

## Gotchas (verified the hard way)

- ament_python console scripts install to `lib/<pkg>/` (`ros2 run`
  location), never on `PATH`. If a CLI ever needs to be on `PATH`,
  also install a shim via `data_files` `('bin', [...])` — colcon then
  emits a `path.sh` hook for the package.
- README quickstarts lead with the one-command tasks, then point at
  `pixi task list` instead of duplicating command tables.
