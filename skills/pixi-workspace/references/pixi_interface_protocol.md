# Pixi Workspace Interface Protocol

|                |                                                                    |
|----------------|--------------------------------------------------------------------|
| **Title**      | Pixi Workspace Interface Protocol                                  |
| **Type**       | Process (convention)                                               |
| **Status**     | Ratified 2026-07-05 (see [Status](#status))                        |
| **Created**    | 2026-07-05                                                         |
| **Applies to** | All Berkeley Humanoids pixi workspaces (see [Scope](#scope))       |

## Abstract

This document defines how every Berkeley Humanoids pixi workspace exposes
its environment, tasks, and scripts. It establishes a three-level command
model:

1. a **universal lifecycle core** of five reserved pixi tasks
   (`setup`, `build`, `test`, `clean`, `doctor`),
2. a **packaged product toolbox** (`hc`) for commands whose meaning is
   invariant across workspaces, and
3. a **scenario-task vocabulary** (`sim`, `real`, `policy`, …) whose names
   are reserved but whose semantics each workspace defines.

It further specifies a one-command day-one ritual (`pixi run build`),
rules for authoring tasks, and an environment-activation protocol. The
goal is that anyone who has learned one workspace can operate any other,
and that the canonical ROS 2 commands keep working verbatim underneath.

## Status

**Ratified and implemented 2026-07-05.** This copy is preserved with the
`pixi-workspace` skill for traceability of the design rationale. The
durable normative text lives in the dev tree's `AGENTS.md` →
"Environment, tooling & version control" → "Command interface (three
levels)"; the public walkthrough in
`Humanoid-Control-Website/docs/how_to/use_pixi_tasks.md`; the
operational rules in the sibling `SKILL.md`. Implemented via
[humanoid_control#16](https://github.com/Berkeley-Humanoids/humanoid_control/pull/16),
[Lite-Deployment#1](https://github.com/Berkeley-Humanoids/Lite-Deployment/pull/1),
[Humanoid-Control-Website#13](https://github.com/Berkeley-Humanoids/Humanoid-Control-Website/pull/13),
plus the local `humanoid_control_ws` manifest (30 → 19 tasks). The
migration plan in
[Appendix A](#appendix-a--migration-map-non-normative) is retained as
the historical record of that change.

## Scope

This protocol governs:

- **`humanoid_control_ws`** — the development workspace,
- **the `*-Deployment` repositories** — operator workspaces,
- **the `humanoid_control` repo-root buildfarm manifest** — packaging
  (a maintainer-CI surface, partially exempt; see
  [Lifecycle tasks](#3-level-1--lifecycle-tasks)).

There is no official pixi task style guide beyond this document; it sets
convention rather than follows one. Its choices were informed by the
official pixi/RoboStack guidance, a survey of ~15 real pixi + ROS 2
projects (ros2/ros2, NVIDIA IsaacSim, NASA-JSC, ros-controls, bit-bots,
RobotecAI, rerun, …), and the uv ecosystem's day-one conventions. Key
findings from that survey are cited inline as rationale.

Throughout, **must**, **must not**, **should**, and **may** are used in
the RFC 2119 sense.

## Audiences and design goals

Three audiences must find the interface natural:

- **uv-native Python developers** expect the ritual `uv sync` →
  `uv run <thing>`, one runner verb for everything, and a project's
  invariant commands shipped *with the package* as `[project.scripts]`
  entry points (mjlab's `train` / `play`). Pixi maps 1:1:
  `pixi install` → `pixi run`, and `pixi run <exe>` falls back to any
  executable on the environment `PATH` exactly like `uv run train`.
- **ROS-native developers** expect `colcon build`,
  `ros2 launch pkg file.launch.py`, `key:=value` launch arguments, and a
  sourced overlay. All of that must keep working verbatim inside
  `pixi shell` / `pixi run --`; tasks and CLI verbs are shortcuts over
  canonical commands, never a replacement layer.
- **Robot operators** need a short, memorable, self-documenting menu:
  one health check, one command per everyday scenario.

## When to deviate

Reserved names carry the weight of this protocol: a reserved name
**must not** be repurposed, and its reserved semantics **must not** be
altered. Beyond that, the protocol is deliberately thin:

- Workspaces **may** add archetype-local tasks (see
  [§3.1](#31-archetype-local-extras)) provided each carries a
  `description` and is never required of other workspaces.
- The canonical `ros2` / `colcon` commands are part of the interface
  (see [§8.5](#85-escape-hatches)); when no task fits, documenting the
  canonical command is preferred over inventing a task.
- If following a rule would clearly make a specific workspace worse,
  deviate — and record why in the task description or workspace README
  so the deviation reads as a decision, not an accident.

## 1. The three command levels

Every command a user types belongs to exactly one level:

| Level | Commands | Mechanism | Defined by | Scope |
|---|---|---|---|---|
| **1. Pixi-ROS lifecycle** | `setup` `build` `test` `clean` `doctor` | pixi tasks; names **and** semantics reserved by this protocol | each workspace's `pixi.toml` (re-declared — pixi tasks cannot ship with a dependency) | any pixi-ROS repo — ours match the ecosystem's de-facto canon |
| **2. Product toolbox** | `hc <verb> [<noun>]` — `bus`, `motor`, `viz`, `calibrate` | packaged executable on the environment `PATH` | `humanoid_control_cli` | every environment that installs the stack (source or binary) |
| **3. Workspace scenarios & glue** | `sim` `real` `policy` (+ qualifiers), `teleop`, archetype extras | pixi tasks; scenario **names** reserved, **semantics** workspace-defined | that workspace's `pixi.toml` only | that workspace |

### 1.1 The sorting criterion

The rule that separates levels 2 and 3:

> **Invariant meaning → packaged `hc` verb.
> Workspace-dependent meaning → workspace task under the reserved
> vocabulary.**

`hc bus ping` or `hc viz urdf` mean exactly the same thing in every
workspace and for binary-channel consumers — packaging them once
prevents per-tool alias sprawl and drift (two such aliases were broken
in the dev workspace when this protocol was drafted). `sim` or `policy`
legitimately mean different things per workspace (base bringup in the
dev workspace, the full task stack in a deployment) and typically wrap
workspace-local composition — so each workspace declares them, and the
protocol standardizes only their names and grammar.

*Rationale.* In practice the scenario tiers of our workspaces barely
overlap (the dev workspace carries the Prime/piano variants; each
deployment carries its own glue), so the re-declaration cost is nominal
— a few TOML lines per workspace, with a reference block in the docs to
copy from. This is also the faithful reading of the mjlab model: mjlab
ships `train`/`play` — invariant utilities — as `[project.scripts]`; it
does not ship "bring up my specific project" commands. That glue is
always your own project file. Pixi offers no mechanism for tasks that
travel with a dependency — `[tasks]` are strictly
workspace-manifest-level — so the only portable mechanism is the one uv
uses: executables shipped by the package (conda `bin/`, Python console
scripts), which `pixi run` resolves after task names.

## 2. The day-one ritual

Day one, in every developer-facing workspace, **must** be one command:

```sh
pixi run build     # auto-installs the env, fetches sources, colcon-builds
pixi run doctor    # (recommended) verify; prints what to fix if unhealthy
```

### 2.1 How it collapses to one command

- `pixi run` self-heals the environment (installs from the lockfile if
  needed). `pixi install` remains an optional explicit step, kept in
  the docs for uv users who expect a `sync` verb.
- `setup` **must** be safe to chain from `build`, achieved by two
  mechanisms together (both verified empirically):
  - **`vcs import --skip-existing`** — exits 0 on re-run, and unlike
    plain `vcs import` it never resets a locally-checked-out branch in
    a third-party repo back to its pin.
  - **pixi `inputs`/`outputs` caching** — because `--skip-existing`
    alone still *fetches* every remote (and exits 1 offline; a bare
    chain would put the network in front of every build and make an
    offline robot unable to rebuild):

    ```toml
    [tasks.setup]
    cmd = "vcs import src --skip-existing --input humanoid_control.repos && touch build/.setup-stamp"
    inputs = ["humanoid_control.repos"]
    outputs = ["build/.setup-stamp"]
    ```

    Verified semantics: cache hit (no network) when nothing changed;
    re-runs on a `.repos` edit; re-runs after `clean` or on a fresh
    clone (the stamp lives under `build/`).

  Caveat (**must** be documented on the task): `--skip-existing` never
  moves an existing checkout to a new pin — after a `.repos` pin bump,
  delete that repo directory and re-run `setup`. This is no worse than
  the pre-protocol behavior.

### 2.2 Nothing else auto-builds

The chain stops at `build → setup` by design. **No scenario task or
`hc` verb may ever auto-build** — hardware bringups must not trigger
surprise rebuilds. `doctor` is the staleness/health check instead, and
colcon owns build incrementality.

### 2.3 The `doctor` check

`doctor` **must** check: workspace root, environment active, overlay
built and sourced, `/opt/ros` contamination (fail hard),
`ROS_DOMAIN_ID`, source tree present, key packages visible. Fixable
findings are warnings; fatal findings exit non-zero.

## 3. Level 1 — lifecycle tasks

These five names are the only **fully** reserved names — reserved in
both name and semantics. Every developer-facing workspace **must**
provide them (with the `test` exception noted):

| Task | Meaning |
|---|---|
| `setup` | fetch/prepare sources beyond pixi itself; **idempotent**; chained by `build` |
| `build` | build the workspace default lane (depends on `setup`) |
| `test` | run the workspace's tests (omit only if a workspace truly has none) |
| `clean` | wipe `build/ install/ log/`; **never** touches `src/` or `.pixi/` |
| `doctor` | workspace health check (§2.3) |

*Rationale.* The de-facto standard task core across every credible
surveyed project is `build` / `test` / `clean` — kebab-case, flat;
nobody namespaces. Workspace-type repos keep tasks to lifecycle only;
demo/deployment repos add scenario tasks. prefix.dev's only official
task-set prose adds `lint` and a `check` aggregate, which this protocol
treats as archetype-local (§3.1) rather than universal.

### 3.1 Archetype-local extras

Allowed, described, never required of other workspaces:

- **Dev workspace:** `build-all` (include the EtherCAT/Prime lane),
  `test-result` (singular, matching the colcon verb), `lint` (the
  `.githooks` ament linter set), `check` (`test` + `lint` pure alias;
  the local pre-push gate — its description notes local linter versions
  ≠ CI), `gen-dds`, `test-dds`.
- **Buildfarm (maintainer CI, exempt from the developer core):**
  `setup` (aggregating hidden `_import` + `_overlay`), `package`
  (build `.conda` artifacts — deliberately **not** `build`, ending the
  collision with the colcon verb), `publish`.

Convenience variants that duplicate an existing mechanism **must not**
be added. Deleted at migration: `build-pkg` (equivalent to
`pixi run build --packages-select <pkg>` via trailing-argument
forwarding), `test-lint`, `test-results`, `launch-policy-tracking`.

## 4. Level 2 — the `hc` toolbox

### 4.1 Verb set

| Verb | Meaning |
|---|---|
| `hc viz [viser\|rerun]` | live state viewer of a running robot (default: viser) |
| `hc viz urdf` | static URDF/kinematic inspector (sliders + RViz), no robot needed |
| `hc calibrate` | calibration bringup (writes `calibration.yaml`) |
| `hc bus ping\|discover\|probe\|probe-report` | CAN diagnostics |
| `hc motor slider` | MIT-mode slider GUI |

Behavioral requirements:

- Bare `hc` prints the menu with one-line descriptions.
- A verb whose target package isn't installed **must** fail with a
  clear message naming the package.
- Every verb `execvp`s into the canonical `ros2 run` / `ros2 launch`
  command — signals and `key:=value` arguments pass straight through.
- The verb set is fixed, versioned, and CI-tested with the code it
  wraps.

Vocabulary notes: there is no separate `view`/`visualize` verb
(confusable with `viz`), and "model" was rejected as a noun (collides
with the ONNX policy model). The docs site keeps teaching the canonical
`ros2 …` forms alongside `hc`.

### 4.2 Scope

Lite product-stack utilities and shared diagnostics only — **zero
references to anything outside that core**: no Prime, no task
scenarios, no sibling repos.

> **Litmus test:** *identical meaning in every workspace?*
> If not, it is a workspace task.

### 4.3 Installation requirement

`hc` **must** be on `PATH` in any activated environment, source-built
or binary, and `ros2 run humanoid_control_cli hc` **must** keep
working.

*Implementation note.* ament_python puts console scripts in
`lib/<pkg>/` (the `ros2 run` location), never on `PATH` — the root
cause of the previously broken `hc` task. The fix, verified
end-to-end: additionally install a `bin/` shim via `data_files` —
colcon generates a package-level `path.sh` environment hook whenever a
package installs a `bin/` directory, so the shim is on `PATH` after
sourcing — and have the conda recipe place it in `$PREFIX/bin`. The
executable bit was verified under `--symlink-install` (our standard);
confirm it survives a non-symlink colcon install at implementation
time.

### 4.4 Extension mechanism (deferred)

If a future invariant genuinely needs to ship from another package, the
mechanism of record is an ament-resource-index manifest (`hc_commands`
resource type) registered by the owning package. This is **deferred**
until a real case exists; nothing uses it at ratification, and the CLI
carries no extension machinery until then.

## 5. Level 3 — scenario tasks

### 5.1 Grammar

Scenario tasks follow **`<scenario>[-<qualifier>]`**, kebab-case, with
**at most one qualifier**. The reserved scenario words below **must**
be used by every workspace that has the concept; `launch-*` /
`deploy-*` prefixes **must not** be used:

| Scenario | Meaning (workspace-defined scope) |
|---|---|
| `sim` | bring up this workspace's primary stack in simulation |
| `real` | same, on hardware |
| `policy` | prepare + load the RL policy against a running stack |

The unqualified name is the workspace's default target; the suffix
selects the **primary task variant only**. Every secondary parameter
(robot, backend, checkpoint, scene, …) **must** remain a ROS launch
argument, forwarded verbatim — task names must never stack qualifiers.
With a robot × task × backend × policy matrix:

```sh
pixi run sim-piano robot:=prime policy:=latest    # right
pixi run sim-prime-piano-latest                   # wrong — name explosion
```

Which axis is "primary" is the workspace's call (in the dev workspace
today it is the robot or scene: `-prime`, `-piano`); everything else
rides the launch-argument surface, which is exactly what `ros2 launch`
users expect and what §7 forwarding preserves. The task `description`
**must** state the scope and the main launch arguments.

Examples — dev workspace: `sim`, `real`, `sim-prime`, `real-prime`,
`sim-piano`, `policy`, `policy-piano` (unqualified = Lite).
Lite-Deployment: `sim` / `real` are the full reach-task stack
(`launch/task_reach.launch.py backend:=…`) plus `teleop` for its
repo-local gamepad script.

Names outside the reserved vocabulary **may** be used for concepts the
vocabulary doesn't cover (`teleop`), in free kebab-case.

### 5.2 Size cap

The scenario tier **should** stay under roughly a screenful. A launch
file gets a task only if it is an *everyday* entry point; everything
else is documented in canonical `ros2 launch` form, which always works
(§8.5).

## 6. Choosing where a new entry point goes

Apply the first rule that matches:

1. **Invariant meaning in every workspace** (diagnostic, viewer,
   calibration) → `hc` verb. Never a workspace task.
2. **Scenario or workspace-local composition/script** → task in that
   workspace's `pixi.toml`, using the §5.1 reserved vocabulary where it
   applies (`sim`, `real`, `policy`), free kebab-case otherwise
   (`teleop`).
3. **Operates on the workspace** (fetch, build, verify) → lifecycle
   task, universal names only.
4. **Needs root or mutates host state** (CAN buses, kernel) →
   `scripts/*.sh`, run explicitly with `sudo`, never a task.
5. **Rarely used** → nothing; document the canonical `ros2 …` command.

## 7. Task authoring rules

### 7.1 Every public task has a description

`pixi task list` is the workspace menu; `hc` is the toolbox menu. A
task without a `description` is invisible to onboarding.

```toml
# Correct:
sim = { cmd = "ros2 launch <pkg> <file>.launch.py", description = "Bring up the Lite stack in MuJoCo" }

# Wrong: no description — invisible in `pixi task list`
sim = "ros2 launch <pkg> <file>.launch.py"
```

Plumbing tasks are prefixed with `_` (hidden from the menu).

### 7.2 Launch-wrapping tasks are arg-less strings

Trailing `key:=value` arguments must forward verbatim:

```sh
pixi run sim enable_gamepad:=false
pixi run real wandb_run_path:=…      # in a deployment
```

Typed pixi `args` (defaults/choices) are reserved for non-launch tasks
where they collapse variants; they **must not** appear on tasks whose
users pass launch arguments.

### 7.3 Chaining and caching

- `build` chains `setup` (idempotent, §2.1); **nothing else
  auto-chains**.
- No pixi `inputs` caching on `build` — colcon owns incrementality.

### 7.4 Task strings are legible

The task string shows the canonical command it wraps; non-obvious flags
get a why-first comment above the task. Style flags belong in
`COLCON_DEFAULTS_FILE`, not the task string (§8.3).

### 7.5 Forbidden names

- **No `install` or `shell`** — they shadow pixi's own verbs.
- **No `run` or `start`** — `pixi run run` reads badly, and the
  scenario vocabulary already names what starts.
- **No colon namespacing** (`build:all`).
- **No same-name tasks in multiple environments** — the interactive
  picker breaks CI.

```text
# Correct:   setup  build  sim  sim-prime  teleop  _import
# Wrong:     install  shell  run  start  build:all  launch-mujoco  deploy-real
```

## 8. Environment and activation protocol

### 8.1 Overlay sourcing

Every colcon workspace **must** source its overlay through a guarded
activation script (NASA-JSC pattern):

```toml
[activation]
scripts = ["scripts/activate.sh"]
```

```sh
# scripts/activate.sh — sourced by pixi on every run/shell
if [ -f "$PIXI_PROJECT_ROOT/install/setup.sh" ]; then
  . "$PIXI_PROJECT_ROOT/install/setup.sh"
fi
```

POSIX `setup.sh`, not `setup.bash`. The guard makes the first,
pre-build activation a no-op instead of an error.

### 8.2 `[activation.env]` carries the workspace identity

- Deployment workspaces pin their robot's `ROS_DOMAIN_ID` (Lite = 5 per
  the lab guide; Lite-Specialist = 6). The dev workspace leaves it
  unpinned, and `doctor` prints it.
- `RCUTILS_COLORIZED_OUTPUT=1` and the standard
  `RCUTILS_CONSOLE_OUTPUT_FORMAT` go in every workspace.

### 8.3 Colcon flags

Style flags move to `COLCON_DEFAULTS_FILE`
(`config/colcon-defaults.yaml`: `--symlink-install`, compile-commands
export) so plain `colcon build` inside `pixi shell` behaves exactly
like `pixi run build`. Semantic selection flags
(`--packages-skip-regex`, `--packages-up-to`) stay visible in the task
string — policy, not preference.

### 8.4 Channels and environments

- Channel order: `https://prefix.dev/robostack-jazzy` before
  `https://prefix.dev/conda-forge`, full URLs. Binary-consumer
  manifests put `https://prefix.dev/berkeley-humanoids` first.
- Single default environment. Adopt features/environments only when
  dependencies genuinely diverge (the same criterion as the Tier-2 rule
  in AGENTS.md).

### 8.5 Escape hatches

Escape hatches are part of the interface and **must** be documented in
every README:

- `pixi shell` — all `ros2`/`colcon` commands and `hc` work verbatim,
- `pixi run -- <any command>`,
- the canonical `ros2 launch` forms for anything without a task.

### 8.6 The `scripts/` directory

`scripts/` at the workspace root holds host-side helpers that can't run
inside the environment or as tasks: `activate.sh`, `canup.sh` (sudo),
`doctor.sh`.

---

## Appendix A — Migration map (non-normative)

Transitional; describes how existing repos reach compliance. Delete
once executed.

### A.1 `humanoid_control_cli` (small; ships first)

- Add the `bin/` shim install (source overlay) plus `$PREFIX/bin`
  placement in the conda recipe so `hc` is on `PATH` in any activated
  environment (§4.3).
- Add the `viz urdf` noun (targets
  `humanoid_bringup_lite view_lite.launch.py`); keep `bus`, `motor`,
  `viz`, `calibrate`. No scenario verbs, no extension mechanism (§4.4).
- Update the not-found error message to suggest `pixi run build`.

### A.2 `humanoid_control_ws/pixi.toml` (30 tasks → 19)

| Current | New | Note |
|---|---|---|
| `setup` | `setup` | add `--skip-existing` + `inputs`/`outputs` stamp caching (§2.1); gains description |
| `build`, `build-all`, `clean`, `gen-dds`, `test`, `test-dds` | unchanged names | `build` gains `depends-on = ["setup"]`; all gain descriptions |
| `test-results` | `test-result` | colcon verb parity |
| `build-pkg`, `test-lint` | **deleted** | trailing args / replaced by `lint` + `check` |
| — | `lint`, `check`, `doctor` | new; `lint` requires factoring the `.githooks/pre-commit` linter list into a script with a full-tree mode (the hook itself lints staged files only) — uncrustify/cppcheck stay excluded locally per the hook's version-skew rationale |
| `launch-mujoco` / `launch-real` | `sim` / `real` | scenario vocabulary |
| `launch-mujoco-prime` / `launch-real-prime` | `sim-prime` / `real-prime` | |
| `launch-mujoco-piano` / `launch-policy-piano` | `sim-piano` / `policy-piano` | |
| `launch-policy` | `policy` | |
| `launch-policy-tracking`, `launch-midi-keyboard` | **deleted** | back-compat alias; rarely used (canonical form documented) |
| `launch-viz`, `view`, `calibrate` | **deleted** | → `hc viz` / `hc viz urdf` (fixes the dead `bar_description_lite` target) / `hc calibrate` |
| `robstride-*` (4), `mit-slider-gui`, `rerun-viz`, `viser-viz` | **deleted** | already under `hc bus` / `hc motor` / `hc viz` |
| `hc` | `hc = "ros2 run humanoid_control_cli hc"` | fixed forwarding task (kept for pre-`PATH`-shim compatibility; drop once the bin shim ships) |
| `canup.sh` (root) | `scripts/canup.sh` | + new `scripts/activate.sh`, `scripts/doctor.sh` |

Final dev-ws task list: `setup build build-all test test-result lint
check clean doctor gen-dds test-dds sim real sim-prime real-prime
sim-piano policy policy-piano hc`.

### A.3 `Lite-Deployment/pixi.toml` (9 tasks → 7)

| Current | New | Note |
|---|---|---|
| `setup`, `build`, `doctor` | unchanged | `setup` gains `--skip-existing` + stamp caching; its `fetch_rqt_controller_manager.sh` step needs an idempotency guard (`test -d … \|\|`) since `build` chains it |
| — | `clean` | universal core |
| `deploy-mujoco` / `deploy-real` | `sim` / `real` | full reach-task stack via `task_reach.launch.py` |
| `launch-mujoco`, `launch-policy` | **deleted** | debugging compositions; canonical `ros2 launch` forms documented |
| `launch-teleop` | `teleop` | repo-local script, stays a task |
| `visualize` | **deleted** | → `hc viz urdf` |
| — | add `RCUTILS_*` to `[activation.env]` | |

`Lite-Specialist-Deployment` gets the same mapping (its scenario tasks
keep their `backends:=` launch-arg pattern; `ROS_DOMAIN_ID` stays 6).

### A.4 `humanoid_control` repo-root buildfarm manifest

| Current | New |
|---|---|
| `import` + `overlay` | `_import` + `_overlay`, aggregated by `setup` |
| `build-all` | `package` |
| `publish` | unchanged |

**CI is a consumer of these names:** `.github/workflows/build_jazzy.yml`
runs `pixi run import` / `overlay` / `build-all ${{ matrix.arch }}` /
`publish` — update the steps to `setup` / `package <arch>` / `publish`
in the same PR (the trailing arch argument forwards unchanged).

### A.5 Documentation

- `Humanoid-Control-Website` `how_to/use_pixi_tasks.md`: rewrite around
  the three levels (workspace tasks via `pixi task list`; toolbox via
  `hc`), including a copy-paste reference block of scenario tasks for
  new/consumer workspaces; fixes the stale `bar …` mapping and
  `src/humanoid_control/bar.repos` path. Update task names in
  `installation.md`, `lite_101.md`, `quick_reference.md`,
  `troubleshooting.md`, `packages.md`.
- `AGENTS.md`: fold §1–§8 into "Environment, tooling & version
  control"; update decision-log rows (Appendix B).
- READMEs (`humanoid_control_ws` config repo, `humanoid_control`,
  `Lite-Deployment`): lead with the §2 one-command ritual.

## Appendix B — Decision log entries (for AGENTS.md)

| Choice | Picked | Why (vs the alternative) |
|---|---|---|
| Command levels | L1 lifecycle tasks (reserved names+semantics) / L2 packaged `hc` toolbox / L3 scenario+glue tasks (reserved names, local semantics) | invariant-meaning commands are packaged once and cannot drift (two aliases broken today); workspace-flavored scenarios stay in the workspace that defines their meaning; scenario tiers barely overlap across workspaces so duplication is nominal. vs packaging scenarios into `hc` (needs an extension mechanism + scope policing; `sim`'s meaning varies per workspace) or convention-only with no CLI (regenerates per-tool alias sprawl). |
| `hc` scope | fixed toolbox: `bus`, `motor`, `viz [viser\|rerun\|urdf]`, `calibrate`; litmus = identical meaning everywhere; ament-index `hc_commands` extension mechanism deferred until a real cross-package invariant exists | core stays universally reusable, carries no external payload (no Prime/piano). |
| Universal task core | `setup build test clean doctor` only | matches the ecosystem's de-facto canon; everything else is archetype-local. vs an 11-verb reserved set (over-standardized). |
| Day-one ritual | one command: `pixi run build` (env auto-installs; chained `setup` is `--skip-existing` + stamp-cached on the `.repos` file) | uv-like self-healing; fewer steps to memorize; cache keeps the network out of daily/offline builds (a bare chain would fetch every remote per build and fail offline — verified). vs install→setup→build→doctor sequence. |
| Build freshness | `build`→`setup` chain only; scenario tasks and `hc` verbs never auto-build; `doctor` for health | colcon owns incrementality; no surprise rebuilds before hardware bringup. |
| Overlay activation | guarded `scripts/activate.sh` sourcing `install/setup.sh` | explicit first-build handling, POSIX, matches NASA-JSC/ros-controls. |
| Colcon flags | style flags in `config/colcon-defaults.yaml`, selection flags in task strings | manual `colcon build` == `pixi run build`; policy stays visible. |
| Buildfarm verb | `package` (not `build*`) | packaging ≠ compiling; ends the `build-all` double meaning. |
