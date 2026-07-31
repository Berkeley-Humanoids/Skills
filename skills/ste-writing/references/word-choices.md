# Word choices for software documentation

Read this file when you choose substitutions or fix terminology drift.
The principle behind every entry: pick the plainest common word that
keeps the exact meaning, then use that word the same way everywhere.

## Substitutions

| Write | Not |
|---|---|
| use | utilize, leverage, employ, make use of |
| start | commence, initiate, kick off, spin up |
| stop | cease, halt, wind down |
| make sure that | ensure, verify, confirm, check that (unless "verify" names the defined action) |
| before | prior to, ahead of |
| after | subsequent to, following (as a preposition) |
| about (a topic); approximately (a quantity) | regarding, concerning, with regard to, "about 2 GB" |
| get | obtain, acquire, retrieve (unless it names a defined API action) |
| show | display, demonstrate, surface |
| help | facilitate, assist |
| also | additionally, furthermore, moreover, in addition |
| but | however (mid-sentence), that said, having said that |
| thus | so, therefore, consequently, as such, hence |
| to | in order to |
| if | in the event that, should you |
| because | due to the fact that, owing to |
| many | numerous, myriad, a plethora of, a variety of |
| examine; do a check of | inspect, "check" as a verb in strict mode |
| send | dispatch ("transmit" is correct for a signal or energy) |
| necessary; must ("Python 3.10 is necessary") | need as a verb, require, necessitate |
| for example | e.g. |
| that is | i.e. |

## The API's name wins

The substitution table never overrides a defined technical name. If the
platform, standard, or API uses the term, use that term and only that
term:

- `terminate()` exists → write "terminate the process".
- The RFC says "octet" → write "octet", not "byte", when the distinction
  matters.
- Kubernetes says "Pod" → write "Pod", capitalized, every time.
- The CLI flag is `--verbose` → "verbose output" is the correct name for
  what it produces.

Define the term once at first use if a general reader will not know it.

## One name for one thing

These pairs drift constantly in software docs. Pick one term per
document (usually the one your API or platform uses) and hold it:

| Pick one of | Note |
|---|---|
| repository / repo | Match the surrounding docs. Do not alternate. |
| directory / folder | "Directory" for CLI and code contexts, "folder" for GUI contexts. Do not mix in one document. |
| parameter / argument | These are different things. A parameter is in the signature. An argument is the value you pass. Use both, each correctly. |
| flag / option / switch | Pick one for CLI docs. |
| terminal / shell / console / command line | Pick one for "the place you type commands". |
| machine / host / server / node / box | Pick the one your platform defines. |
| user / client / caller | In API docs these can be three different parties. Define which is which, then hold the mapping. |
| endpoint / route / API | An API has endpoints. Do not call one endpoint "an API". |
| package / library / module / dependency / crate / gem | Use the ecosystem's own word. |
| config / configuration / settings | Pick one noun. "Config" is acceptable if used consistently. |
| install / set up | "Install" puts software on the system. "Set up" makes it ready to use. Keep them distinct. |
| error / exception / failure / fault | Match the language's own model (Python raises exceptions, Go returns errors). |
| delete / remove / destroy | Match the API verb. |
| field / property / attribute / key | Match the language and format (JSON has keys, Python objects have attributes). |

## Words that are fine

Plain-language lists sometimes over-ban. Keep these when they are the
precise term:

- **read / write** — approved plain verbs. **create / delete / update**
  are correct when they name the defined operation (the API's name
  wins); otherwise prefer "make" and "erase".
- **must / can / do not** — the clearest modality words. Replace
  "should" with "must" when the requirement is real; a bare "should"
  leaves the reader unsure whether the action is optional. (If the
  document declares RFC 2119 keywords, follow RFC 2119 exactly.)
- **deprecated, idempotent, atomic, thread-safe** — technical terms with
  one meaning. Define at first use in beginner-facing docs.

## Marketing and filler to delete on sight

seamless, seamlessly, robust, powerful, cutting-edge, effortless,
world-class, next-generation, revolutionary, blazing, lightning-fast,
elegant, delightful, turnkey, best-in-class, state-of-the-art,
game-changing, battle-tested, enterprise-grade, supercharge, unlock,
unleash, empower, "it is important to note", "it should be noted",
"it is worth noting", "please note that", "as you can see",
"simply", "just" (as minimizers), "easily", "obviously", "of course",
"In conclusion", "In summary" (in short documents), "Whether you're X
or Y", "look no further".