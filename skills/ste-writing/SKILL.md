---
name: ste-writing
description: >-
  Write or rewrite software documentation in Simplified Technical
  English. Use for READMEs, API docs, manuals, tutorials, docstrings,
  code comments, errors, changelogs, CLI help, and other technical prose.
  Use when the user asks for plain English, short sentences, active voice,
  consistent terminology, clarification, simplification, or de-slopping.
---

# STE Writing

Write technical prose in a controlled plain style. Make the text easy to
read, translate, and parse without losing technical precision. This skill is
based on ASD-STE100 Simplified Technical English: https://asd-ste100.org/.

## Protect source text

Do not change these items unless the user asks:

- Code, identifiers, function names, class names, and keywords
- Commands, flags, environment variables, and file paths
- Code samples and sample output
- Version strings, URLs, link targets, and anchors
- Searchable quoted error strings
- Front matter keys, directives, admonition syntax, and license text

Do not silently change published headings. A heading change can break links
or cross-references. List proposed heading changes separately.

Preserve all facts, constraints, caveats, warnings, and edge cases. Do not
invent facts, benchmarks, examples, or version numbers. Keep precise wording
when simpler wording would change the meaning.

## Select a mode

Use **strict** for instructions and operational text, such as procedures,
install steps, runbooks, warnings, error messages, and CLI help.

- Apply all rules.
- Keep each instruction to 20 words or fewer.
- Expand contractions.

Use **flavored** for explanatory or reporting text, such as README
introductions, tutorials, concept pages, comments, changelogs, release notes,
and PR descriptions.

- Apply the same core rules.
- Treat 25 words per sentence as a target, not a hard limit.

If the user gives no mode, use strict for instructions and flavored for other
text. A document can use both modes.

## Core rules

### Terminology

- Use one name for one thing. Do not alternate synonyms for style.
- Use the API, standard, or platform term when it defines the name.
- Give each word one meaning. Use words in their exact technical sense.
- Prefer short common words when they preserve the meaning. See
  `references/word-choices.md` for substitutions.
- Define unfamiliar terms at first use.
- Keep noun clusters to three words or fewer when practical.
- If a long name repeats, define a shorter form or abbreviation at first use.
- Avoid slang, loose jargon, and unnecessary phrasal verbs.
- Prefer verbs over nominalizations: "validate the input", not "perform
  validation of the input".
- Use gender-neutral language.
- Remove marketing claims, empty intensifiers, filler transitions, padded
  summaries, and empty closing text.
- Use American English unless the house style says otherwise.

### Verbs and sentences

- Use active voice and name the actor when the actor is known.
- Do not use passive voice in instructions. In description, use passive voice
  only when the actor is unknown or unimportant.
- Prefer imperative, simple present, simple past, or simple future.
- Avoid perfect tenses and stacked auxiliary verbs when a simple form works.
- Avoid `-ing` as the main action when a direct finite verb is clearer.
- State the primary action directly: "Seal the port with a firewall rule",
  not "Use a firewall rule to seal the port".
- Write short, concrete sentences. Replace vague claims with checkable facts.
- Put a condition before the command: "If the port is in use, change the
  `--port` value."
- Do not remove subjects, verbs, or articles only to shorten a sentence.
- Replace ambiguous pronouns or `this` with the noun.
- Keep `that` when its removal can reduce clarity.
- Hyphenate multi-word modifiers when needed for one clear reading.
- Do not use semicolons. Write two sentences.

### Lists and procedures

- Use a numbered list for a sequence of three or more steps.
- Put one action in each step and use the imperative form.
- Keep actions together only when they must occur at the same time.
- Put an immediate limit or result directly after the action it constrains.
- Use a colon before a list. Keep list items grammatically parallel.
- Do not mix instructions and descriptive facts in one list.
- In a safety list, repeat `do not` in each applicable item.
- After an action that changes visible state, state the expected result.
- Use `must` for hard requirements, safety conditions, or descriptive
  requirements. Use the imperative for normal procedure steps.
- A note gives information only. Put required actions and limits in steps.

### Explanations

- Introduce one new idea at a time.
- Use one topic per paragraph, with at most six sentences.
- Start a paragraph with a topic sentence when the format allows it.
- Repeat key terms instead of using stylistic synonyms.
- Use the same wording for recurring statements and actions.

### Warnings

Use **Warning** for irreversible outcomes, such as data loss or security
exposure. Use **Caution** for recoverable damage. Use the higher level when
both apply.

Start with the command or condition, then state the consequence:

`**Warning:** Back up the database before you run the migration. The migration deletes the legacy tables.`

## Sentence length

For sentence limits, count each of these as one word:

- Code span, flag, path, or identifier
- Number with its unit
- Quoted string, title, or proper noun
- Parenthetical phrase
- Hyphenated group

A list lead-in is one sentence. Each list item is another sentence.

## Code documentation

The ecosystem's doc-comment convention overrides the preferred summary form.
Apply the STE rules to the remaining prose.

| Ecosystem | Summary form | Example |
|---|---|---|
| Python (PEP 257) | Imperative | "Return the parsed config as a dict." |
| Rust | Third-person descriptive | "Parses the config file at `path`." |
| JSDoc / TSDoc | Third-person descriptive | "Parses the config file at the given path." |
| Javadoc / KDoc | Third-person verb | "Returns the parsed configuration." |

For parameter and return descriptions, use fragments if the house style uses
them. Keep the form consistent.

For inline comments, explain why instead of restating the code. Use full
sentences for block comments. Short fragments are acceptable for end-of-line
comments. Keep `TODO(owner):` and `FIXME:` forms unchanged.

## Documentation sites

- Keep front matter, MDX imports, directives, and admonition markers unchanged.
  Edit only their prose.
- Use link text that names the target. Do not use generic text such as `here`.
- Follow the site's document model and house style when they exist.
- Tables can use consistent fragments when full sentences add no value.

## References

Read `references/doc-types.md` when the task matches one of its document
types. It contains format-specific rules for READMEs, API references,
tutorials, how-to guides, errors, CLI help, changelogs, and commit or PR text.

Read `references/word-choices.md` when you must choose a simpler word, resolve
terminology drift, or check whether a technical term should stay unchanged.

## Workflow

1. Read the complete source before you edit it.
2. Record the facts, constraints, warnings, commands, links, and structure that
   must remain.
3. Select strict or flavored mode for each section.
4. Rewrite only the prose that the user asked you to change.
5. Compare the result with the source and restore any lost substance.
6. Check terminology, actors, sentence length, references, and protected text.
7. Return only the requested deliverable. Add an audit table only if the user
   asks for one.

## Final check

Before you return the text, make sure that:

- Each sentence adds information or directs an action.
- One term names each thing throughout the text.
- Every pronoun has one clear referent.
- Known actors are named.
- Every fact, warning, constraint, and qualifier remains.
- Notes contain information only.
- Strict-mode instructions are within the sentence limit.
- Protected code, paths, flags, links, and identifiers are unchanged.
