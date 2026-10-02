---
name: simplify-codebase
description: Review, plan or execute a cleanup pass over an existing codebase - collapsing duplication, splitting files that do more than one thing, simplifying APIs, replacing hand-rolled machinery with something already installed, and deleting what is dead or unused. Use when the user asks to simplify, clean up, refactor broadly, reduce duplication or test count, "make this less of a mess", consolidate storage or abstractions, remove unnecessary gadgets, asks for simplification ideas or a broad "make the codebase simpler" audit, or asks for a refactor spec. Not for adding features, and not for hunting bugs.
---

# Simplifying a codebase

The goal is fewer concepts, not fewer characters. A pass that removes 500 lines and adds two new indirections has lost.

## Principles

- Programmer time is expensive. Optimize for simplicity, interpretability and elegance over cleverness, configurability, or preserving historical structure.
- Favor the design that is easiest to explain to a new maintainer, and fewer concepts over more generic ones.
- Favor explicit dominant paths over symmetric but rarely used alternatives.
- Move complexity to one well-named boundary rather than spreading it across helpers.
- Transform before erasure: do cleanup, filtering, truncation and policy decisions while data is in its richest typed form, keep one canonical structured representation as long as possible, and serialize or adapt it at one clear boundary. Derive multiple output views from that representation instead of post-processing rendered data.
- Keep type distinctions that encode real invariants or valid states. Merge types only when the differences are decorative.
- Use `assert` for internal invariants and impossible states, exceptions for failures, and structured return types for interfaces likely to grow.
- If the software is prerelease or headed for a major revision, do not preserve backwards compatibility by default (section 13).

Work in this order. Each step is cheap and makes the next one smaller.

## 1. Read before proposing

Read the whole thing if it is under about 5000 lines. Do not sample. Half the findings in a real cleanup come from noticing that two files solved the same problem differently, and you cannot notice that from a grep.

Past that size, fan out read-only subagents by directory or subsystem, keep the shared/core layer yourself, and verify every claim they return before you repeat it. Subagents over-claim, especially about dead code.

Read the project's own docs first - `CLAUDE.md`, `spec/`, `docs/`, `ADR/`. They usually record which apparent redundancies are deliberate. Removing a "duplicate" that a doc explains as a security boundary is the worst outcome available.

## 2. Measure before believing

A few commands beat an hour of reading:

```sh
wc -l $(find . -name '*.py' -not -path './.venv/*') | sort -n     # where the mass is
git log --since="90 days ago" --pretty=format: --name-only | sort | uniq -c | sort -rn | head
```

Churn is the strongest signal there is. The file that is touched most often is almost always the file doing too many jobs, and it is the one whose split pays.

Then check the actual state on disk. Table sizes, file counts, log volume. A spec that says "540 KB" and a file that is now 2.1 MB is the difference between a tidy-up and a real problem.

## 3. Check the dependency tree before adding a dependency

Before proposing any library, find out what is already installed transitively.

```sh
uv run python -c "import candidate; print(candidate.__version__)"   # already there?
rg -A6 '^name = "the-package"' uv.lock                              # what does it pull in?
```

Big frameworks drag in general-purpose libraries. An ORM, an HTTP client, a driver, a serializer - one of them is often already sitting in the venv because something else needed it. Choosing that one costs nothing and is usually the right answer even if it is not the trendiest.

When you do compare candidates, compare on: what it costs to install, whether it matches the codebase's existing async/sync shape, and whether its distinctive feature is one this codebase actually uses. A library whose selling point is unused is a library you are paying for and not using. Check current versions on the registry - your knowledge of releases is stale.

## 4. Find duplication by shape, not by name

Grep for the pattern, not the identifier. The same idea reimplemented three times rarely shares a name.

Things that duplicate silently:

- **Connection/session helpers.** `def _db()` in four modules, each with its own schema string.
- **Error-to-text decorators.** Two files each define a private `_tool` that is the same eight lines with a different prefix.
- **Registration boilerplate.** Three calls to the same scheduler API, each restating the same options and the same reasoning in a docstring.
- **Encodings of one type.** Timestamps as ISO strings here, unix floats there, native datetimes in a third place. Pick one.
- **Indirection records with one implementation.** A dataclass of callbacks that only one concrete provider ever fills should become direct calls on that provider. A transaction handed around as a callback should become a context manager.
- **Boilerplate that restates a declaration.** Snapshot, projection or serialization functions that list a dataclass's fields again, and settings subclasses that each repeat what a base class could hold. Derive them from the fields.
- **Repeated keyword arguments in declarative registries.** Test cases, routes or jobs that all pass the same handful of arguments. Move them into a constructor default. This is often the largest production win in a registry-heavy codebase.
- **Cross-module private imports.** `from other_module import _private_thing` is the loudest possible signal that the thing lives in the wrong file. Grep for imports of underscore names across package boundaries and treat every hit as a finding.

## 5. Local simplifications

Inside functions, classes and files:

- Fewer branches, temporary variables and state mutations. Early returns instead of nested conditionals. Direct data flow instead of hand-rolled plumbing. Drop single-use variables where that reads better.
- Delete stub functions that only rename another call, forward one line, or hide no policy. Delete one-off helpers that exist only to preserve an old layering choice.
- Extract an object when state and behavior travel together, when the same bundle of parameters repeats, when lifecycle logic spreads across call sites, or when an invariant is enforced informally.
- Collapse duplicate parsing, normalization, validation or formatting into one path.
- Replace boolean- or sentinel-heavy control flow with enums, clearer objects, or distinct code paths.
- Replace open-coded guard branches for impossible states with `assert`. Raise instead of returning ad hoc error objects or strings through normal result paths when the caller already handles failure.

## 6. Structural simplifications

- **API surface.** Fewer arguments, fewer optional knobs, fewer overlapping entrypoints. Two pathways for the same action (an implicit-target variant and an explicit-target one) usually fold into one.
- **Special cases.** Fold repeated special cases into one mechanism when the abstraction is real, such as several inline behaviors moved into one dispatch path.
- **Gadgets.** Code for a feature or future that is no longer intended, compatibility layers nobody relies on, extension points with no concrete users, and caches, planners, wrappers or managers that add indirection without paying rent. See also section 9.
- **Parallel representations.** The same concept stored in several shapes, dual APIs for one action, or separate metadata and runtime paths that could be unified.
- **Split ownership.** Too many layers touching the same state, as when a builder, a registry, a loop and a helper each partly own one behavior.
- **Speculative abstractions.** Plugin systems, strategy hooks, config knobs, generic wrappers or adapters built for flexibility the product does not need. If two paths exist and one is clearly preferred, delete the other rather than abstracting over both.
- **Mini-frameworks.** Prefer the standard library or direct data structures when custom machinery adds maintenance without leverage.
- **N calls in a loop.** Repeated subprocess, HTTP or database calls inside a loop over results, especially when the count is unbounded or driven by user data. One batched call plus a membership check is the simplification target even when the loop is correct.

## 7. Count call sites before extracting

Duplication is not automatically worth removing. Three call sites of a five-line block do not need a module; they might not need a function.

Extract when the duplicated thing carries a **decision** - a policy, a default, a security rule - because then the copies will drift and one of them will drift wrong. Leave it alone when it is just shape.

Never create an abstraction with one implementation. If there is one caller today and a second is "coming", wait for the second: it will define the interface better than your guess will.

## 8. One file, one goal

Split a file when it holds two jobs that a reader has to hold in their head at once, not merely when it is long. The reliable test: can you write the module docstring in one sentence without "and"?

Signals a file has too many jobs:

- Other modules import private names out of it (see above).
- It is the top of the churn list.
- Its imports span layers - a transport library, a UI framework and a database in one file.

When you split, prefer moving *out* the piece that does not belong to the file's name. `cos/bot.py` holding telemetry setup means telemetry moves out, not that `bot.py` gets carved into six.

Watch for dependency direction. If moving a helper would make a low-level module import a high-level one, invert instead: pass the value in as an argument rather than letting the callee fetch it. That usually simplifies both sides. But check every edge before committing to the inversion -- if three other functions in the same module also reach across, inverting one of them fixes nothing and the helper simply belongs at the lower layer.

**A file with mutable module-level globals cannot be split as a move.** Trace who writes them and who reads them first. If the writer and readers land in different modules you have either a cycle or a rewrite: every bare-name read becomes attribute access at call time, because the binding is rebound after import. The usual resolution is a third leaf module owning the globals and importing nothing local. Draw the import graph before promising anyone this phase is mechanical.

Split the module's *tests* with it. A test file larger than the module it covers is a sign the module had several jobs all along, and leaving the test whole undoes half the split.

## 9. Delete before you abstract

Ranked by value: delete > move > collapse > abstract.

Before claiming anything is dead, search the whole tree including tests, string-keyed lookups, config files, and entry-point declarations. Then distinguish three cases, because they have different answers:

- **Dead**: no consumers anywhere. Delete it, and name the tests that go with it - green tests are not an argument for keeping code.
- **Specified but unwired**: no consumers, but a design doc says where it goes. Either wire it (often ten lines) or delete it and the doc. Leaving it is the worst of the three.
- **Used only by tests**: the test is keeping a corpse warm. Delete both. The exception is a guard against malformed author input, such as a constructor that rejects an invalid declaration: it protects the next author, not runtime. Flag it for a decision instead of deleting it.

Two more cases are not dead, but often cost as much:

- **Reachable but unused capabilities.** The code is imported and wired, but no shipped configuration, case, schedule or caller exercises it: an optional backend left blank in every config, an authoring mode no shipped case uses, injection points for callers that do not exist. Find these by reading configs, registries and CLI usage, not imports. Retiring one is a product decision, so list it under approval with the behavior that goes away.
- **Finished migrations and fallbacks.** Cutover commands, compatibility readers, and fallbacks that mint a value when configuration is missing often outlive the transition they served. State the precondition for deleting each one, such as the old schedule drained or the key configured everywhere, and ask the user whether it holds.

## 10. Shrinking the test suite

Consolidating tests and cutting the test count are different jobs. Shared fakes, decorators and table-driven tests cut lines but rarely cut collected tests, and parameterizing can raise the count. If the user wants fewer tests, run a separate deletion survey. Every proposed deletion must name the retained test that still covers its contract, and risky deletions get flagged. Expect about 10%, not 30%.

Kinds of tests to delete:

- **Vacuous parameter rows.** These are rows whose assertion never runs, for example a matrix over every registered case where most cases have nothing to check. Count the rows where the guard short-circuits.
- **Restatements.** Snapshots of constants, static checks that repeat a declaration, schema-layout and dataclass-field snapshots, and tests of upstream library or language behavior.
- **Helper twins of an integration test.** When an end-to-end test already drives a helper through its real caller, delete the helper-level test of the same decision. This is usually the most common deletion.
- **Lexical variant enumeration.** Seven spellings of one rejection reduce to one representative plus each distinct boundary, such as overflow or whole-string matching. Security fences are the exception: keep every adversarial input that exercises a distinct path.

Kinds of consolidation that cut lines:

- Nested `async def run()` bodies inside sync tests become a shared coroutine-test decorator.
- Fakes duplicated across test files move into one support module with a unique basename.
- Tests of a pure library function move into that library's own package. Keep only the integration checks in the consumer.

## 11. Trimming prose

Comments that narrate the adjacent code, record history, or repeat rationale already kept in a design-decisions document can go. A prose-only pass is cheap and often removes as many lines as a whole deduplication stage.

Keep provider quirks, security and replay reasoning, idioms at system boundaries, and caveats about future limits ("spool this to a file if it grows"). Trimming workers delete exactly those caveats unless told otherwise, so spot-check the diff. Model-visible strings are specifications, not prose: tool docstrings, prompts, field descriptions, error text. Never trim them for redundancy.

## 12. Prefer structure over strings

The highest-value data-model fixes are almost always these:

- A column encoding two facts in one string (`"done:tag-a+tag-b"`). Split it.
- The same object stored half as columns and half as a JSON blob, with a hand-written function to decide which fields go where. Store the object.
- Loose `dict[str, Any]` crossing a boundary where the caller immediately reads three known fields. The same goes for `Record<string, unknown>`, `object` or `any` at a response boundary with a stable shape.
- Tuples with positional meaning, or stringly encoded multi-purpose payloads, where a dataclass or typed record would name the fields.
- One omnibus record whose fields are not valid for every case. Prefer a discriminated union or specific classes.

Do the transformation while the data is still typed, and serialize once at the edge.

## 13. When there is nothing to preserve, say so out loud

If the project is prerelease, or the real data lives somewhere else, then migrations, compatibility shims and dual-read paths are pure cost. Confirm it once with the user, write it at the top of the plan, and then take the clean break everywhere - including deleting migration code the old design had already specified.

Be precise about what "wipe it" means. `ls` the directory and check for a nested `.git` before recommending its deletion: derived caches and irreplaceable user data are frequently siblings. Name the exact files to delete, never the parent directory.

Watch the test suite's hermeticity while you are moving state around. Suites usually redirect one root path in a fixture, and everything safe is *derived* from it. A new config field spelled as its own literal escapes that redirect, which points the tests at production -- so derive it, the way the existing safe fields are derived. Check whether the fields that already exist are as safe as you assume; one of them usually is not, and it is the one that deletes files.

## 14. Two structural patterns worth hunting for by name

**A callback that cannot carry what it must record.** When a producer hands work to a consumer and the consumer has to write down that it did the work, check whether the callback signature can even name the thing. If it cannot, the usual instinct is to widen the signature. The better move is often to invert it: let the consumer *poll* for its own work, selecting rows by the state it maintains. It then has the identity by construction, and the hand-off, its ordering constraints, and any separate catch-up sweep all disappear together. Serialize with whatever "one worker" mechanism the codebase already has.

Two things to get right when you invert: the predicate almost always needs to exclude work that is still being written, and the "find my work" query has to be captured as one recorded read if the runtime replays it, or a retry sees a different set.

**A producer awaiting a result it discards.** Grep for `await` on calls whose return value is unused or immediately dropped. Each one is a caller blocking on work it does not need, and the cost compounds when the producer holds a serialized resource - a poll loop, a lock, a queue with concurrency 1. Converting these to queued background work is usually a small change with a large latency payoff.

Two cautions. Background it *durably*, using whatever the runtime offers for recoverable child work - firing a bare in-memory task converts a slow path into a lossy one. And check for the deliberate exception: some synchronous calls exist so that a downstream consumer cannot observe a half-built state, and their docstrings usually say so. One counterexample is what keeps this from becoming "queue everything".

## 15. Run the library. Do not reason about it.

Every plausible-sounding claim about a library's behaviour is a coin flip until you execute it. In one pass over a small codebase, four separate API assumptions that "obviously" held were false:

- a column type declared timezone-aware silently returned naive values on one backend, so every comparison would have raised;
- an enum type documented as producing a constraint produced none without a second keyword;
- an upsert's "did it already exist" answer came back as `-1` on one backend and `1`/`0` on the other, which would have silently dropped every record;
- a two-argument scalar function existed on one backend and only as an aggregate on the other.

Each was a one-paragraph script to settle and each would have been a production bug. Write the throwaway script. Print the emitted SQL, the round-tripped value, the actual DDL. If a claim spans two backends or two versions, test both -- that is precisely where the difference lives.

The same goes for the framework you already depend on: read its source for what it keys on. Durable-execution and queue libraries commonly resume on a *positional ordinal* as well as a name, which means adding or reordering a step breaks in-flight work exactly as badly as renaming one -- a rule you will not find by grepping for names.

## 16. Changing a representation breaks its sentinels

When you replace a type, hunt its sentinel values before anything else.

- A float or int timestamp used as a falsy zero (`if not self._last`) becomes always-truthy as a `datetime`. The code keeps running and quietly means something else.
- A stringly-typed column becomes an enum, so **enumerate every value ever written to it first**, across the whole tree. Miss one and a CHECK constraint turns a rare path into a hard failure -- and the one you miss is always the rare path, which is why nobody remembered it.
- An empty collection reaching a query builder can emit valid SQL on one backend and a syntax error on another.

## 17. Respect the invariants

Some duplication is load-bearing. Before moving or merging anything, find out what enforces the project's rules:

```sh
rg -l 'ast\.|assert not|is_relative_to|monkeypatch' -g 'tests/**'
```

Tests that parse the source, assert on paths, or check that a module never reaches a given call are encoding a property that a refactor can silently break. Name them in the plan, say exactly how each one has to be re-pointed, and make "this test still fails when the property is violated" one of the acceptance checks.

The same goes for framework rules that key on names - durable-workflow engines, task queues, migration tools and plugin registries often resume or dedup on a function or job name. Moving a function between modules is safe; renaming it is not. State the rule once and keep every such name byte-identical through the pass.

## 18. Write the plan as phases that each leave the tests green

A cleanup that lands as one commit will not be reviewed. Order the phases so the mechanical ones come first:

1. **Pure moves.** No behaviour change, no schema change. Easiest to review, and it shrinks everything after it.
2. **The structural change.** One subsystem at a time.
3. **The specs that were waiting on it.**
4. **Deletions and collapses.**

Say what each phase costs and what could go wrong. Then add a **Considered and not doing** section listing everything you noticed and rejected, one line each on why. Proposals that usually belong there:

- A shared runner or framework across packages whose setup genuinely differs. The common part shrinks to a few lines once the differences are kept.
- Removing an alternate output format or representation that looks unused. It is often deliberate, so ask.
- Replacing a hand-written UI widget with a library when the widget enforces app-specific constraints or keyboard behavior. The adapter and the retained accessible controls eat most of the savings, and a build step and payload are added.
- Collapsing thin wrapper pairs that exist as test seams, for example a durable entry point around a plain body. That section is what stops the reader from re-raising the same ideas, and it stops you from quietly dropping something inconvenient.

## 19. Have it attacked before you build it

A cleanup plan is a design document and deserves the same treatment. Run adversarial review on separate axes, in parallel, each with a narrow brief:

- **Technical**: is every factual claim in this true? Verify against the installed libraries and the real files, not from memory.
- **Over-engineering**: which of these proposed modules does not earn its existence? Which two should be one?
- **Invariants**: what does this silently break?

Give each reviewer the standard the plan should meet and forbid it from commenting on prose. Keep each earlier draft; a rejected idea with a recorded reason is worth more than a clean file.

**Then verify what they return.** Reviewers over-claim in both directions, and a confident file:line citation is still capable of being wrong. In one pass a reviewer asserted a package was already an installed dependency when it was only an optional extra, and another counted references to two files that had no references at all. Both would have propagated into the plan as fact. Check anything load-bearing yourself before folding it in.

Expect reviewers to disagree with each other, and treat that as signal rather than noise: one round said "merge these two tiny modules into the existing runtime module", the next round showed that doing so would make importing any tool require an API key. The second round exists to catch the first round's corrections.

**Watch for right conclusions with wrong reasons.** Several findings will be correct while the argument for them is not -- a "duplicate" that differs in one load-bearing line, a "live bug" that cannot actually fire, a justification that cites a spec saying the opposite. Fix the reasoning even when the conclusion stands, because the reasoning is what the next reader inherits.

And do not assert a negative you have not checked. Writing "I read these three files and they were fine" is padding unless you read them; when someone does read them, the claim usually turns out to be false, and it costs the whole document its credibility.

## 20. Running a large pass with worker agents

Past roughly 20k lines, survey and edit through parallel workers, and keep verification and commits in the parent.

- **Ask for write permission up front.** An auto-mode permission classifier may block write-mode agent launches such as `codex exec --dangerously-bypass-approvals-and-sandbox`. Read-only surveys usually pass, for example `codex exec -s read-only -C . -o report.md - < prompt.md`. Ask for write permission before the plan is done, not when the first stage is waiting.
- **Survey by area.** Use core, services, evals and tests, one read-only worker each. Save the reports to the scratchpad. Every later worker prompt points at them, so no worker re-derives context.
- **Use one shared worker preamble.** It covers the repository rules and an exclusive file list per worker. It forbids `git stash`, checkout, reset, add and commit, and forbids editing the plan file. It also requires a test-coverage check at the end, ruff, pyright and the affected tests, and a final report: status per item, files touched, and the measured `git diff --numstat`.
- **Partition workers by file ownership, not by plan item.** Two items that touch one file go to the same worker. A shared helper that needs another worker's file gets skipped and reported.
- **The parent verifies the combined tree.** Run the full suite, the type checker and pre-commit. Remove the landed items from the plan, and commit once per stage.
- **Write wait loops carefully.** Poll in a background loop on report files. Never call `exit` inside an `until` condition, because it kills the loop on the first check. `pgrep -f` matches its own command line.

### Plan shape that worked

- **One plan file.** It holds Constraints, behavior-preserving stages in order, "Needs approval" and "Recorded decisions". Remove each item when it lands. Then the file shows exactly what is left.
- **Stage order.** Dead code first, then production dedup, test consolidation, prose trimming, and finally the test-count cut.
- **The user annotates items inline,** for example `HUMAN: APPROVED` or `HUMAN: DENIED`. Turn a denial into a one-line Recorded decision so nobody raises it again. Record skipped items the same way, for example "saves 2 lines; not worth it".
- **A deletion that depends on a pending decision goes under that decision,** not into the current stage.

### What paid, and what did not

- **Prose trimming is cheap and large.** It cut about 2k lines in one pass. Rationale comments get shortened along with the rest, so spot-check that security, replay and provider-quirk reasoning survived. Model-visible strings, such as tool docstrings, prompts, field descriptions and error text, are specifications and stay untouched.
- **Survey estimates overshoot.** Expect some items to shrink to almost nothing. Workers should skip and report them rather than force them through.
- **Test consolidation cuts lines but not the test count.** Parameterizing can even raise the collected count. If the user wants fewer tests, run a separate deletion survey. Every deletion must name the retained test that still covers its contract, and risky deletions get flagged. A realistic cut is about 10%, not 30%.
- **Hunt vacuous parameter rows by name.** These are rows whose assertion never runs, for example a matrix over every case where most cases have nothing to check. One such matrix produced 74 of the deletions.
- **Watch worker drift.** Workers add regression tests when they change a seam, and they give support modules names that record history, such as the plan stage that created them. Check numstat for test growth and rename such modules.
- **Price a library-replacement proposal before recommending it.** Measure first-party lines saved, payload, the build step it adds, and whether it enforces the existing constraints. A UI library that cannot do the app's keyboard editing saves far less than the widget's line count suggests.

### Proving behavior is unchanged

- **Declarative registries,** such as eval cases, routes and settings: dump every instantiated object before and after the change, then diff the dumps.
- **Comment-only and prose-only edits:** compare the AST with docstrings stripped, and compare the model-visible strings separately.
- **Serializers and projections:** compare byte-identical output, including key order, over real fixtures.

### Reporting

To report a delta, split `git diff --numstat master HEAD` by package, and within each package into code, tests (paths under `tests/`) and other files. Show added, removed and net for each. Say which commits the comparison covers.

## Review questions

Ask these while scanning:

- Is this abstraction carrying real policy, or only indirection?
- Are there two ways to do the same thing?
- Can this special case fold into the default path?
- Would deleting this code make the system easier to explain?
- Is this state owned in one place?
- Does this helper preserve an old design the current product no longer needs?
- Is this API built for a hypothetical future instead of present usage?
- Is this return type weaker than the actual shape warrants?
- Is this loop making N external calls where one call and a membership check would do?

## Output

If the user asked for a review or a plan, write it. If they asked for the work, do the work.

For a review or plan:

- Findings first, ranked by impact. Simplification opportunities, not style nits. Separate quick wins from structural changes, and call out deletions explicitly.
- Every finding gets: what is too complex, why it is that way today, the simpler shape, the migration cost and risk, and file references.
- End with the **Considered and not doing** section (section 18): one line on what was seen and one on why it was excluded. Nothing noticed is silently dropped.
- Keep the prose plain. A refactor spec is read by someone deciding whether to spend a week, and ornament costs them time.

When a review turns into implementation, turn the highest-value findings into the phased plan of section 18.

A survey prompt for a worker, to adapt:

> Review <area> for opportunities to simplify it, both local (simpler functions, stub functions with no policy, state and behavior that belong in one object) and structural (API surface, special cases that fold into a general mechanism, gadgets, obsolete extension points, duplicated pathways, speculative abstractions, split ownership, compatibility layers, dead or unused code). Read the project's docs first; they record which redundancies are deliberate. For each finding give what is too complex, the simpler shape, why it is safe, the estimated net lines for code and tests, the file references, and the migration risk. Mark findings that change observable behavior. Do not edit files.
