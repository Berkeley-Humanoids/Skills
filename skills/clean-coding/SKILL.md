---
name: clean-coding
description: Enforce minimal, direct code when writing or reviewing: no provenance comments, speculative config/state, wildcard signatures, argument bloat, one-use helpers, dead branches, or unnecessary documentation. Prefer concise names, centralized real configuration, linear code flow, and the smallest scoped diff.
---

# Clean coding

Write the smallest obvious implementation that satisfies the current requirement. Unnecessary generality is a bug. Every added line needs a concrete reason to exist now.

Follow established project conventions for mechanical style. Do not make unrelated edits.

## Keep the implementation direct

- Reuse existing code and the standard library before adding abstractions, dependencies, files, or configuration.
- Do not add flexibility, extension points, defensive branches, or settings for hypothetical future needs.
- Prefer a simple data representation that removes special cases over adding conditionals around them.
- Preserve existing behavior and interfaces unless the task requires changing them.

## Names

Use the shortest conventional name that is unambiguous at its scope. Names describe what a thing is, not its history or implementation story.

- Prefer concise domain names; do not turn identifiers into English sentences.
- Spell words out. Use an abbreviation only when it is standard in the domain or already conventional in the codebase (`cmd`, `idx`, `ms`); never invent contractions or drop vowels. Write `_left` / `_right`, not `_l` / `_r`.
- Include units or reference frames when ambiguity matters: `timeout_ms`, `pos_world`.
- Do not encode change history or recency: no `new`, `old`, `v2`, `final`, `fixed`, or similar suffixes unless they are the actual domain distinction.
- Do not introduce a variable merely to name a simple expression used once unless the name materially improves clarity.

## Functions and control flow

- Every parameter and return value must be necessary and used. Remove unused arguments, single-value flags, and redundant conditions.
- Use explicit signatures. Do not use a bare `*` keyword-only marker. Avoid `*args` and `**kwargs`; use them only when an interface you do not control requires forwarding them.
- Validate at the boundary. Do not repeat checks internally for states that cannot occur.
- Prefer straightforward control flow and eliminate special cases when the data or structure can make them ordinary cases.
- Do not extract a private helper merely to shorten a function. Inline one-use helpers. Extract when logic is reused by another code path or an interface requires a separate callable.
- Long function bodies are fine when they represent one readable procedure. Use sparse section comments to divide major steps instead of scattering the flow across helpers.

## State and configuration

- Do not introduce module/global state for convenience when a local, parameter, or object attribute is sufficient.
- Do not turn a stable implementation constant into configurable state merely to make it adjustable.
- Real user/project configuration belongs in the project's existing centralized config location. If none exists, create one organized config module/file only when there are enough genuine settings to justify a configuration surface.
- Do not scatter config values across unrelated implementation files.

## Comments and docstrings

Code is self-explanatory by default. Write a comment only for:

1. a section header in a long procedure;
2. a non-obvious pitfall or easy-to-miss correctness issue;
3. an enduring constraint, invariant, unit/frame convention, protocol rule, or rationale.

Never restate obvious code. Never turn conversation, debugging, or diff history into source-code provenance: no "changed from", "previously", "for now", "after debugging", or similar narration.

If an old decision contains an enduring fact, keep only the fact:

`// Raised from 100 Hz after debugging` -> `// Controller diverges below 200 Hz.`

Otherwise delete the comment. History belongs in the issue, commit, PR, or design document.

Docstrings are brief API overviews, not walkthroughs. State only purpose and non-obvious contract details such as units, frames, side effects, ownership, or exceptional behavior. Omit docstrings when the name, signature, and types already say enough. Do not add `Args:` / `Returns:` blocks that merely repeat the signature.

## Dead code and structured config

- No commented-out code, superseded branches, unused parameters, or compatibility paths kept "just in case". Version control holds history.
- Delete `from __future__ import annotations` unless the module needs deferred evaluation, such as annotations that reference `TYPE_CHECKING`-only imports.
- In JSON/YAML/TOML/URDF/MJCF and similar files, add only fields consumed by the program, defined by the schema, or requested by the user.
- Never invent metadata such as `_note`, `reason`, `changed_from`, `old_value`, or `migration_note` to preserve conversation history.

## Before done

Read the diff as if you had never seen the conversation. Remove any added comment, helper, argument, condition, config field, global, file, or abstraction that is not required by the resulting system.
