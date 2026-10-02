---
name: terse-comments
description: Write or tighten a comment or docstring in this repo - state the decision, then only what the code cannot say. Use when writing a new comment or docstring, when asked to make one shorter or less wordy, or when one argues its case instead of stating it. Applies on top of ste-writing, which governs sentence form.
---

# terse-comments

Expect to cut a third to a half of what you first write. That ratio is a smell, not a target: a comment already made of nothing but distinct facts is finished, and cutting to hit a number takes one of them.

## First: is it agent-facing?

A docstring on a function registered as an agent tool is the model's spec, and so is any string that reaches a prompt. Apply sentence form only. Never delete anything there for being redundant: a tool docstring that restates an argument is doing its job, and the enumerated options, examples and limits are the behavior. If shortening would drop one, leave the sentence long.

Everything below this section is for text a human reads. When you cannot tell which you are looking at, treat it as agent-facing.

## Docstrings

**Line 1 names the thing.** A noun phrase beats a sentence about what the function does: "Static welcome for an unregistered account, at most once per day" over "The one static line an unregistered account gets each day, or nothing". Then any behavior that looks like a bug and is not, stated flatly as intended.

**Then one paragraph of what the code cannot say.** External API quirks, a consequence with no local symptom, the security reason for a boundary.

A naming line and one paragraph is the whole shape. There is no third paragraph, and suspect the break rather than the content: a third usually holds a real fact that belongs at the end of the second, so merge it rather than dropping it.

## Comments

A comment has no contract paragraph, because it cannot restate a signature. Lead with the decision it explains, then the reason that decision is not obvious. One paragraph, unless two decisions share the site.

The code sits directly below, so "inferable" bites harder than in a docstring. A comment that narrates the next line goes, and so does one whose consequence the reader meets two lines later anyway.

## Cut

- **The justification chain.** "This is intended behavior" beats the scenario that proves it, and an abstract failure rarely needs its worked example.
- **The closing flourish.** A last sentence that re-frames or summarizes what you just said.
- **The contrast clause.** "...the way it could in the workspace", "...rather than the shape that would repeat instead". The claim already stands without it.
- **The pointer that then restates.** "Same reasoning as above", followed by the reasoning. Pick one.
- **Vendor mechanism trivia.** Name the constraint you hit, not the feature matrix you learned it from. A framework's own behavior earns a clause, never a paragraph: "PTB hands an update the raising handler did not stop to the agent path below" survives, and the walk through PTB's group loop does not.
- **The defence of where the code sits.** Why this function and not the applier, why this handler group. Name the collaborator instead -- "handled in `on_code`" -- and see the Keep entry on rejected alternatives.
- **Orientation.** Where a thing lives, which module it borrows transport from, what another file argues. All discoverable.
- **Advice, down to its imperative.** "Reaching a second bot means replacing that global with a registry keyed by target" -> "A second bot needs a registry keyed by target".

## Keep

- External behavior you would have to test to learn: "Vikunja rejects a label filter given a title rather than an id".
- **Why the obvious call was not used, where nothing else records it.** A rejected design belongs in the project's design-decision record (such as `spec/ASSUMPTIONS.md`), so check there first: repeating it in a docstring is the duplication the section below governs. What stays is the rejection a reader would re-make while editing this function, with the failure it would have caused.
- A consequence invisible from the code in front of you: "a silently failed advance would re-raise the job forever".
- The security reason for a boundary. "The inbox must not write the instructions that govern it" survives every cut.
- Exact contract terms: half-open window notation, ordering, units, what is paired with what.
- **A repeated idiom, verbatim at every site.** "Never fatal: a checkpoint that cannot be written must not cost Gaurav his turn" reads as duplication and is not. The repetition is what makes the pattern legible, so never dedupe one to a pointer.

## Two places say the same thing

Keep the one at the higher altitude and cut the other. A fact about the whole module belongs in the module docstring, and a per-function pass that trims it there loses it for good, because the function that also stated it will be trimmed by the same rule on its own turn. The design-decision record is an altitude too, and the highest one.

Altitude is not the only test: keep the copy that is load-bearing where it sits. "Phone, then desktop" explains the excluded second tap in `enrol.epitaph`, where the comparison lives, and is scenery in the handler that calls it.

Check before cutting either copy. This is the one failure in this skill that destroys information rather than merely bruising it.

## Notes

Shorter beats grammatically ideal. Passive voice is fine when it costs fewer words.

Verify before you compress. Shortening a claim re-asserts it, and the pass that produced this skill carried forward a docstring saying labels were re-checked in a function that never looks at them.
