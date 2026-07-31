---
name: ste-writing
description: >-
  Write or rewrite software documentation in Simplified Technical
  English: READMEs, API references, manuals, tutorials, docs-site
  pages, docstrings, code comments, error messages, changelogs, and
  CLI help. Use when asked to write, edit, simplify, clarify, or
  de-slop technical prose, or for plain English, consistent
  terminology, active voice, or short sentences. 
---

# STE Writing

Write software documentation in a controlled plain style that is easy
to read for non-native speakers, easy to translate, easy for other
agents to parse, and free of AI slop. This standard is derived from
ASD-STE100 Simplified Technical English, which you can access at
https://asd-ste100.org/.

## Scope

Apply this skill to:

- READMEs and repository docs
- API reference pages and generated-doc source (docstrings, doc comments)
- Usage manuals, how-to guides, runbooks, install guides
- Tutorial and concept pages on documentation websites
- Inline code comments
- Error messages, log messages, CLI help text
- Changelogs, release notes, deprecation notices

Never change:

- Code, identifiers, function names, class names, keywords
- CLI commands, flags, environment variables, file paths
- Code samples and sample output, even if the prose inside them is poor
- Version strings, URLs, link targets, anchors
- Quoted error strings that users will search for
- Front matter keys, directives, admonition syntax, license text

Be careful with headings in published docs. A changed heading can break
deep links and cross-references. When you edit an existing page, list
proposed heading changes separately. Do not apply them silently.

## Choose a mode

**strict** — for text that tells the reader what to do: procedures,
install steps, how-to guides, runbooks, error messages, CLI help,
warnings. Apply every rule and the 20-word sentence cap.

**flavored** — for text that explains or reports: README introductions,
tutorials, concept pages, code comments, changelogs, release notes, PR
descriptions. The same rules apply, but the 25-word sentence cap is a
target rather than a hard limit, so the text keeps a natural rhythm.

If the user gives no mode and the text tells a reader what to do, use
strict. Otherwise use flavored. A single page can mix modes: strict for
its numbered steps, flavored for its introduction.

## The rules

### Words

- Use one name for one thing across the whole document. Never "repo" in
  one section and "repository" in the next, never "config file" then
  "settings file" for the same file.
- The API's own name wins over plain-word substitution. If the method
  is `terminate()`, write "terminate the process", not "stop the
  process".
- Prefer the short common word: use, start, make sure, before, about,
  also. The full substitution table is in
  `references/word-choices.md`.
- Give each word one meaning and hold it. Use words in their exact
  sense, not their loose one: "the request failed", not "the request
  died".
- No slang or jargon: "the update made the router unusable", not "the
  update bricked the router".
- Do not combine plain words into phrasal verbs: "start the container",
  not "spin up the container"; "delete the branch", not "blow away the
  branch".
- Do not turn nouns into verbs. If "backup" is the noun in your
  terminology, write "create a backup", not "backup the database".
- Keep noun clusters to three words or fewer. Unpack longer stacks with
  prepositions ("the config for the priority handler of the retry
  queue"), or hyphenate the words that act as one unit.
- When a name is longer than three words, write it in full at first
  use, then define and use a shorter form or its abbreviation.
- Define an unfamiliar term at first use. Use gender-neutral language.
- Delete marketing adjectives and empty intensifiers on sight:
  seamless, robust, powerful, blazing-fast, effortless, battle-tested,
  enterprise-grade, "simply", "just".
- Delete filler transitions, padded summaries, and empty closers: "It
  is worth noting that", "In conclusion", "Whether you're X or Y".
- American English spelling, unless the house style says otherwise.

### Verbs

- Use the active voice. Name the actor: "you", the tool, or a named
  component. "Run the installer", "The daemon writes a log".
- Never use the passive in steps. In descriptive text, the passive is
  allowed only when the agent is truly unknown ("during transmission,
  the data was corrupted"). When the agent is merely unstated, recast
  with "you", "we", or the component: "the flag can be set" → "you can
  set the flag".
- Use simple tenses only: imperative, simple present, simple past,
  simple future. No perfect tenses ("the build has failed" → "the
  build failed") and no stacked auxiliaries ("must be configured" →
  "Configure X" in a step, "you must configure X" in description).
- A past participle before a noun states a condition, not an action:
  "the cached response", "the connection is closed".
- Avoid "-ing" main verbs: "When you run the build", not "When running
  the build". The "-ing" form survives in technical names and headings
  ("the logging system", "Getting started").
- Express an action as a verb, not a nominalization: "validate the
  input", not "perform validation of the input".
- Name the primary action verb: "Seal the port with a firewall rule",
  not "Use a firewall rule to seal the port".

### Sentences

- Write short, concrete sentences. Replace an abstract claim with a
  checkable one: "cache size affects performance" → state the
  direction, and give the numbers if you have them.
- Put the condition before the command, separated by a comma: "If the
  port is in use, change the `--port` value."
- Do not omit words to shorten a sentence. Keep subjects, verbs, and
  articles: "If enabled, the proxy..." → "If the flag is enabled, the
  proxy...". "Files not backed up" has two parses; "the files that are
  not backed up" has one.
- In strict mode, expand contractions: "do not", not "don't". In
  flavored mode, contractions may stay if they match the house voice.
- Keep the conjunction "that" after verbs like "make sure", "show", and
  "means": "Make sure that the service is stopped."
- When a pronoun or "this" can point to two things, replace it with the
  noun.
- Keep articles, but not before a noun followed by an identifier:
  "port 8080", "line 42", "step 3".
- Hyphenate multi-word modifiers: "read-only mode", "long-running
  process".
- No semicolons. Write two sentences.
- Connect related sentences with plain connectors — "and", "but",
  "then", "thus", "as a result" — and with demonstratives: "This
  method...".

### Lists

- Use a numbered list for a sequence of 3 or more steps, one action per
  item, imperative form.
- End the lead-in with a colon. Make every item parallel and
  grammatically connected to the lead-in.
- Do not mix instructions and description in one list.
- In safety lists, repeat "do not" in each item instead of factoring it
  into the lead-in.

### Procedures (strict mode)

- Keep each instruction to 20 words or fewer.
- Write one instruction per sentence. Exception: actions that must
  occur at the same time stay together ("Hold the reset button and
  reconnect the power"). A limit or immediate result also stays with
  its action: "Measure the startup time. It must be under 2 seconds."
- The imperative is the requirement: "Restart the service", not "you
  must restart the service". Reserve "must" for safety text, hard
  conditions, and descriptive requirement statements.
- After a step that changes visible state, tell the reader what to
  expect: "The terminal prints the server URL."
- Notes give information only — never an instruction, requirement, or
  limit. If the reader must act on it, it is a step. If it prevents
  damage or loss, it is a warning. The test: the reader must be able to
  complete the procedure with every note deleted.

### Descriptions (flavored mode)

- Keep each sentence to 25 words or fewer.
- Give information gradually: one new idea per sentence, each building
  on the one before.
- Repeat key words to link sentences. Elegant variation is ambiguity —
  if sentence one says "the scheduler", sentence three does not say
  "the dispatcher".
- Start each paragraph with a topic sentence that names its topic. The
  topic sentences alone should read as an outline of the page.
- One topic per paragraph, at most six sentences.
- Use the same wording for the same kind of statement every time it
  occurs, across the whole document.

### Warnings

- Grade the risk: warning for irreversible outcomes (data loss,
  security exposure, an unbootable device), caution for recoverable
  damage. When the two overlap, use the higher level.
- Start with the command or the condition, never with background. Then
  state the consequence: "**Warning:** Back up the database before you
  run the migration. The migration deletes the legacy tables."

### Counting words

For the sentence caps, count as one word each: a code span, a flag, a
path, an identifier, a number with its unit, a quoted string, a title,
a proper noun, a parenthetical, and a hyphenated group. A list's
lead-in ends at the colon and counts as one sentence, and each item
counts as another.

## In-code documentation

Each ecosystem fixes the form of a doc comment's first line. That
convention wins over the imperative preference above. Apply the rules
to everything else: word choice, sentence length, one meaning per word,
active voice inside the descriptive frame.

| Ecosystem | Summary-line convention | Example first line |
|---|---|---|
| Python (PEP 257) | Imperative | "Return the parsed config as a dict." |
| Rust | Third-person descriptive | "Parses the config file at `path`." |
| JSDoc / TSDoc | Third-person descriptive | "Parses the config file at the given path." |
| Javadoc / KDoc | Third-person, starts with a verb | "Returns the parsed configuration." |

For parameter and return descriptions, use fragments if the house
style uses them ("timeout — maximum wait in seconds"). Keep the
fragment form consistent across every entry.

For inline comments: explain why, not what the code already says. Use
full sentences for block comments. Short fragments are fine for
end-of-line comments. Keep `TODO(owner):` and `FIXME:` conventions
unchanged.

Read `references/doc-types.md` before you write a specific document
type. It covers READMEs, API references, tutorials, how-to guides,
changelogs, error messages, and commit/PR text, with the conventions
that override or extend the rules for each.

## Documentation websites

- Keep front matter, MDX imports, Sphinx directives, and admonition
  markers exactly as they are. Edit only the prose inside them.
- Write link text that names the target: "see the [install
  guide](install.md)", not "click [here](install.md)".
- Follow the site's document model when one exists. In Diátaxis terms:
  tutorials and explanations take flavored mode, how-to guides and
  reference pages take strict mode.
- Tables in reference pages may use consistent fragments instead of full
  sentences.

## Workflow

1. Read the full source before you edit.
2. List the facts, warnings, constraints, commands, links, and structure
   that must survive the rewrite.
3. Select a mode for each section.
4. Rewrite the prose. Do not touch anything on the never-change list.
5. Compare the rewrite with the source. Restore any lost fact, caveat,
   or qualifier.
6. Self-check the rewrite against the Final checks list below. Fix
   every real violation. Identifiers and deliberate style choices are
   not violations.
7. Return only the requested deliverable. Add an audit table (rule /
   before / after) only when the user asks for one.

## Preserve substance

- Do not invent facts, benchmarks, examples, or version numbers.
- Do not remove caveats, limitations, warnings, or edge cases to make
  the text shorter.
- Do not weaken a precise technical claim into a vague plain one. If a
  simplification would lose precision, keep the longer phrasing and flag
  the trade-off.

## Final checks

- Does each sentence add information or direct an action?
- Is each thing called by exactly one name everywhere, and each
  recurring action worded the same way everywhere?
- Does every pronoun have one clear referent?
- Is every actor named where the actor is known?
- Did every fact, warning, and constraint survive?
- Do notes contain information only, with every limit and requirement
  in its step?
- In strict mode, is every sentence within its length cap, with all
  contractions expanded?
- Are all code spans, paths, flags, and links byte-identical to the
  source?
