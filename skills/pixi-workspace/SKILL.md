---
name: pixi-workspace
description: Use when creating, editing, or reviewing pixi.toml manifests, pixi tasks, workspace scripts (activate/doctor/canup), the hc CLI, or any Berkeley Humanoids pixi+ROS2 workspaces.
---

# Pixi workspace guideline

Every Berkeley Humanoids pixi workspace exposes the same three-level
command interface.

1. **General ros2-pixi tasks** - standardized set of procedures for every ROS2-pixi workspace.
2. **Product toolbox `hc`** - packaged CLI on `PATH` containing common robot utility helpers.
3. **Scenario pixi tasks** - common workspace-defined tasks: `sim`, `real`, `policy`, `viz` etc.

Sorting rule for new commands: 
invariant meaning in every workspace → `hc` verb.
workspace-dependent meaning → workspace task.
`hc` carries only the Lite product core + shared diagnostics.
Needs root / mutates host state (CAN, kernel) → a `scripts/*.sh` run with explicit `sudo`, never a task.
Rarely used → no alias; document the canonical `ros2 …` command.

## Level 1 — generic tasks (exist in every workspace)

| Task | Meaning | Rules |
|---|---|---|
| `setup` | fetch/prepare sources beyond pixi itself (`vcs import`, fetch scripts) | idempotent and cached (canonical shape below); safe to re-run; chained by `build` |
| `build` | build the workspace's default libraries with colcon | `depends-on = ["setup"]`; style flags come from `config/colcon-defaults.yaml`, only semantic selection flags (`--packages-skip-regex`, `--packages-up-to`) appear inline |
| `test` | run the workspace's tests, linters excluded | `colcon test … --ctest-args -LE linter`; omit only if the workspace truly has no tests |
| `clean` | wipe the colcon overlay | exactly `rm -rf build install log`; never touches `src/` or `.pixi/` |
| `doctor` | workspace health check (`scripts/doctor.sh`) | checks: workspace root, env active, `/opt/ros` contamination (hard fail), sources imported, overlay built + sourced, `hc` on `PATH`, `ROS_DOMAIN_ID`; warnings for fixable, non-zero for fatal |

Common setup procedure is one command: `pixi run build` (pixi self-heals the env; `setup` is chained and cached), verified by `pixi run doctor`.

Archetype-local extras are allowed (with descriptions), never required of other workspaces — dev workspace: `build-all`, `test-result`
(singular, matches the colcon verb), `lint` (= `.githooks/pre-commit
--all`), `check` (= `test` + `lint`), `gen-dds`, `test-dds`; buildfarm
manifest: `setup` (hidden `_import` + `_overlay`), `package`, `publish`
— the packaging verb is `package`, never `build*`.

## Level 2 — `hc` toolbox verbs (standardized, packaged)

| Verb | What it does |
|---|---|
| `hc bus ping` | single-actuator GetDeviceId ping — no Enable, safe on a powered robot |
| `hc bus discover` | scan a CAN bus for Robstride device ids (read-only) |
| `hc bus probe` | link probe: RTT/jitter (inert) or +lumped delay (PRBS torque) |
| `hc bus probe-report` | RTT distribution + per-DoF lumped delay + plots |
| `hc motor slider` | live MIT-mode slider GUI against one motor |
| `hc viz` | live state viewer of a running robot (`viewer:=viser` default, `viewer:=rerun`) |
| `hc viz viser` / `hc viz rerun` | the standalone live-viewer tools directly |
| `hc viz urdf` | static URDF/kinematic inspector (sliders + RViz), no robot needed |
| `hc calibrate` | calibration bringup; writes `calibration.yaml` |

Every verb `execvp`s into the canonical `ros2 run` / `ros2 launch`
command, so signals and trailing args pass straight through. Bare `hc`
prints this menu. New invariant tool → new verb in
`humanoid_control_cli`, never a per-workspace task alias
(`robstride-ping`, `rerun-viz`, … are banned).

## Level 3 — scenario naming

- Grammar: `<scenario>[-<qualifier>]`, kebab-case, **at most one
  qualifier**, encoding only the primary task variant. All secondary
  parameters stay ROS launch arguments:
  - Right: `pixi run sim-piano robot:=prime policy:=latest`
  - Wrong: `pixi run sim-prime-piano-latest`
- Reserved scenario words: `sim` (primary stack in simulation), `real`
  (same, on hardware), `policy` (prepare + load the RL policy against a
  running stack).
- Unqualified = the workspace default (dev ws: Lite base bringup;
  a deployment ws: its full task stack — state the scope in the task
  `description`).
- Never `launch-*` / `deploy-*` prefixes. No task named
  `install`/`shell` (shadow pixi verbs) or `run`/`start` (redundant
  next to the scenario vocabulary).

## Task authoring

- Every public task carries `description = "…"` — `pixi task list` is
  the menu. `_` prefix hides plumbing tasks.
- Launch-wrapping tasks are arg-less command strings so trailing
  `key:=value` forwards verbatim. Don't use typed `args` on them.
- **No scenario task or `hc` verb may auto-build** (a hardware bringup
  must never trigger a surprise rebuild). Only `build` chains `setup`.
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

## Environment

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
  location), never on `PATH`. To put a CLI on `PATH`, also install a
  shim via `data_files` `('bin', [...])` — colcon then emits a `path.sh`
  hook for the package.
- README quickstarts lead with the one-command ritual, then point at
  `pixi task list` and `hc help` instead of duplicating command tables.
