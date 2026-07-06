---
name: pixi-workspace-interface
description: Use when creating, editing, or reviewing pixi.toml manifests, pixi tasks, workspace scripts (activate/doctor/canup), the hc CLI, or any Berkeley Humanoids pixi+ROS2 workspace (humanoid_control_ws, *-Deployment repos, the buildfarm manifest).
---

# Pixi workspace interface

Every Berkeley Humanoids pixi workspace exposes the same three-level
command interface. Ratified 2026-07-05; durable rationale lives in the
dev tree's `AGENTS.md` ("Command interface (three levels)") and the
public walkthrough in `Humanoid-Control-Website/docs/how_to/use_pixi_tasks.md`.

## The three levels

1. **Lifecycle pixi tasks** — reserved names AND semantics, identical in
   every workspace: `setup` (fetch sources; idempotent, cached), `build`
   (default lane; chains `setup`), `test`, `clean` (wipe
   `build/ install/ log/` only), `doctor` (health check).
2. **Product toolbox `hc`** — a packaged CLI on `PATH`
   (`humanoid_control_cli`): `hc bus …`, `hc motor slider`,
   `hc viz [viser|rerun]`, `hc viz urdf`, `hc calibrate`.
3. **Scenario pixi tasks** — reserved names, workspace-defined scope:
   `sim`, `real`, `policy` (+ at most one qualifier).

Sorting rule for new commands: **invariant meaning in every workspace →
`hc` verb; workspace-dependent meaning → workspace task**. `hc` carries
only the Lite product core + shared diagnostics — never Prime, piano, or
sibling-repo references. Needs root / mutates host state (CAN, kernel) →
a `scripts/*.sh` run with explicit `sudo`, never a task. Rarely used →
no alias; document the canonical `ros2 …` command.

## Scenario naming

- Grammar: `<scenario>[-<qualifier>]`, kebab-case, **at most one
  qualifier**, encoding only the primary task variant. All secondary
  parameters stay ROS launch arguments:
  - Right: `pixi run sim-piano robot:=prime policy:=latest`
  - Wrong: `pixi run sim-prime-piano-latest`
- Unqualified = the workspace default (dev ws: Lite base bringup;
  a deployment ws: its full task stack — state the scope in the task
  `description`).
- Never `launch-*` / `deploy-*` prefixes. Never per-tool task aliases
  duplicating `hc` (`robstride-ping`, `rerun-viz`, …).
- No task named `install`/`shell` (shadow pixi verbs) or `run`/`start`
  (redundant next to the scenario vocabulary). The buildfarm packaging
  verb is `package` — `build*` is reserved for colcon builds.

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
  Semantic selection flags (`--packages-skip-regex`, `--packages-up-to`)
  stay visible in the task string.
- `[activation.env]` always sets `RCUTILS_COLORIZED_OUTPUT=1` + the
  console format; deployment workspaces pin their robot's
  `ROS_DOMAIN_ID`, the dev workspace leaves it unpinned.
- Channels: `https://prefix.dev/robostack-jazzy` before
  `https://prefix.dev/conda-forge` (binary consumers put
  `https://prefix.dev/berkeley-humanoids` first). Single default
  environment; add features/environments only when dependencies
  genuinely diverge.
- No rosdep, no apt — `pixi.toml` is the dependency source of truth.
- `doctor` checks: workspace root, env active, `/opt/ros` contamination
  (hard fail), sources present, overlay built + sourced, `hc` on PATH.

## Gotchas (verified the hard way)

- ament_python console scripts install to `lib/<pkg>/` (`ros2 run`
  location), never on `PATH`. To put a CLI on `PATH`, also install a
  shim via `data_files` `('bin', [...])` — colcon then emits a `path.sh`
  hook for the package.
- Day-one ritual is one command: `pixi run build` (pixi self-heals the
  env; `setup` is chained and cached). Verify with `pixi run doctor`.
- README quickstarts lead with that ritual, then point at
  `pixi task list` and `hc help` instead of duplicating command tables.
