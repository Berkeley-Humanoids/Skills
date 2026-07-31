# Rules by document type

Read the section for the document type you are about to write or edit.
Each section states the mode, the conventions that override the core
rules, and the failure pattern to avoid.

## README

Mode: flavored.

- First paragraph: state what the project is and what problem it solves,
  in 2–4 sentences, no sentence over 25 words, zero marketing
  adjectives.
- Structure to cover, in order: what it is, install, minimal working
  example, where the full docs are, license.
- The minimal example must run as pasted. Test it if you can.
- Failure pattern: the "Whether you're building a chatbot, a pipeline,
  or an agentic workflow..." paragraph. Delete the audience flattery.
  State what the software does.

## API reference

Mode: strict for behavior statements, fragments allowed in tables.

- One entry per symbol. Lead with the summary line in the ecosystem's
  convention (see SKILL.md table).
- Describe behavior in the present tense, active voice, with the symbol
  as the actor: "`flush()` writes the buffer to disk and clears it."
- Document every parameter, the return value, and every error the
  caller can see. Fragments in parameter tables are fine if every entry
  uses the same fragment form.
- State side effects and preconditions as their own sentences. Do not
  bury "must be called after `init()`" in a subordinate clause.
- Failure pattern: describing the happy path only. Errors and edge
  behavior are the reason people open a reference.

## Docstrings and doc comments

Mode: strict.

- The summary-line form follows the ecosystem convention (Python
  imperative, Go identifier-first, Rust/JSDoc descriptive — see the
  table in SKILL.md).
- After the summary line, every rule in SKILL.md applies.
- Follow the house docstring style (Google, NumPy, reST, standard Go
  doc, Rustdoc sections `# Examples` / `# Errors` / `# Panics`). Do not
  convert between styles unless asked.
- Do not restate the signature. "Args: path (str): the path" adds
  nothing. Say what the parameter means and what values are valid:
  "path: Location of the config file. Must exist and be readable."
- Keep doc examples runnable. In Python, doctest-format examples must
  pass.

## Inline code comments

Mode: flavored.

- Explain why, not what. The code states what.
- A comment that paraphrases the next line is a defect. Delete it.
- Full sentences with a capital and a period for block comments.
  Fragments are acceptable at end of line.
- Keep `TODO(name):`, `FIXME:`, `NOTE:`, `SAFETY:`, `# type:`, lint
  directives, and license headers exactly as the codebase uses them.
- When a comment records a decision, name the constraint: "Retry at
  most twice. The upstream rate limit resets after 2 s." Not "be
  careful here".

## Tutorial

Mode: flavored for narrative, strict for the steps themselves.

- Address the reader as "you". The imperative is your default verb form.
- One action per numbered step, ≤ 20 words.
- After every step that changes visible state, state the expected
  result: "The terminal prints `Listening on :8080`."
- State prerequisites (versions, accounts, prior pages) before step 1.
- A tutorial must work every time. Remove choices; a tutorial is not the
  place for "alternatively, you can...". Put alternatives in a how-to
  guide.
- Failure pattern: skipped state. If step 5 works only after an edit the
  reader made in step 2 of another page, the tutorial is broken.

## How-to guide / usage manual

Mode: strict.

- Title names the goal: "Rotate the API keys", not "Key management".
- Numbered steps, one action each, imperative form.
- Put the condition before the command: "If the service is running,
  stop it first."
- State the verification step at the end: how the reader knows it
  worked.
- Warnings come before the step that needs them, marked with the site's
  admonition syntax, in three parts: risk label, command or condition
  first, then the consequence. "**Warning:** Back up the database
  before you run the migration. The migration deletes the legacy
  tables."
- Notes give information only. A limit, requirement, or action belongs
  in the step itself, directly after the action it constrains.

## Concept / explanation page

Mode: flavored.

- One topic per page, one topic per paragraph, ≤ 6 sentences per
  paragraph.
- Define each term at first use. Then use only that term.
- Analogies are allowed. Keep one analogy per concept and drop it once
  the concept is defined.
- Failure pattern: hedged non-claims ("can potentially help improve").
  Make the claim or delete it.

## Error messages and log messages

Mode: strict, hardest form.

- Three parts, in order, as separate sentences: what happened, why (if
  known), what to do.
- Name exact values: the limit, the timeout, the path, the header to
  check. "The request exceeded the 10 MB body limit", not "the request
  was too large".
- No blame ("you provided an invalid..."), no hedging ("something may
  have gone wrong"), no apology, no exclamation marks.
- The message must make sense out of context. It will be read in a log,
  three weeks later, by someone who did not send the request.
- Keep error identifiers and codes stable. Users search for them.

## CLI help text

Mode: strict.

- One-line description per flag, fragment form, same form for every
  flag: "Limit output to N lines", "Write the report to FILE".
- The synopsis line follows the platform convention (`[OPTIONS]`,
  `<required>`, `[optional]`). Do not invent notation.
- Examples section: real commands that run as pasted, most common use
  first.

## Changelog and release notes

Mode: flavored.

- Follow the house format (Keep a Changelog headers Added / Changed /
  Fixed / Removed / Deprecated / Security, or the project's own).
- One line per change. Start with a verb. Name the user-visible effect,
  not the internal refactor: "Fixed a crash when the config file is
  empty", not "Refactored config loading".
- Breaking changes: state what breaks and the exact migration action.
- Link the issue or PR number where the house style does.

## Commit messages and PR descriptions

Mode: flavored.

- Subject line: imperative, ≤ 50 characters, no trailing period.
- Body: what changed and why. The diff shows how; the message records
  why.
- One logical change per commit message claim. If the body needs
  "also", the commit may need a split — say so instead of hiding it.
- PR description: state the problem, the approach, and how you tested
  it, as three short paragraphs or headed sections. No filler openers.
