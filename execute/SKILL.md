---
name: execute
description: Phase 3 of the agentic coding workflow. Executes the approved proposal checklist one task at a time, marking each item complete in the proposal document as it is finished.
---

The user has approved a proposal and wants you to implement it.

## Before starting

Locate the proposal document in `<git-repo-root>/doc/work-sessions/` (use `git rev-parse --show-toplevel`). If which proposal to implement isn't obvious, ask the user.

Read the entire proposal, especially the **Implementation Checklist**, before writing any code.

## Execution rules

- The proposal is an HTML document; each checklist item is an `<li><input type="checkbox"> ...</li>`.
- Work through checklist items **one at a time, in order**. No skipping. No combining steps.
- For each checklist item, follow this exact sequence:
  1. Do the work (write/edit code, run tests, etc.)
  2. **Immediately call the Edit tool to mark that item done in the proposal document by adding the `checked` attribute to its checkbox** (`<input type="checkbox">` → `<input type="checkbox" checked>`). This Edit call must happen in the same response as the work, before any tool calls for the next item. Do not batch mark-offs.
- Do not stop to ask for feedback mid-implementation unless you hit a genuine blocker that requires a decision. Terse corrections from the user ("wider," "wrong file") are fine to act on immediately.
- Run tests continuously throughout. Do not let a phase accumulate failures.
- Write minimal comments — only where the logic is non-obvious.

## Phase boundaries — STOP and wait for commit

Each phase is a separate commit, made by the user.

- After completing the final checklist item of a phase, **STOP**. Do not begin the next phase.
- Report what was finished and explicitly tell the user the phase is ready to commit.
- **Propose a commit message** for the phase. Format it as a fenced code block so the user can copy it directly. Match the repo's existing commit style (run `git log --oneline -20` if you're unsure). Lead with whether the phase is structural or behavioral when the proposal labeled it that way.
- **Wait for the user to commit the code themselves before continuing.** Do not run `git commit` on their behalf, and do not start the next phase until the user signals to proceed (e.g., confirms the commit landed or tells you to continue).
- If the user asks you to keep going without committing, push back once and remind them that each phase is its own commit. If they insist, comply.

## TDD discipline

Follow the red/green cycle prescribed in the checklist:

1. Write or modify the test first
2. Run — confirm it fails for the right reason (compile error is acceptable when production code doesn't exist yet; once it compiles, expect a runtime assertion failure)
3. Write or modify production code
4. Run — confirm tests pass before proceeding

## Tests are the spec — do not rewrite them to fit the code

The test encodes the behavior we want. Production code is what bends.

- **NEVER** rewrite, weaken, delete, or `skip` a test to make it pass against the current production code. If a test fails, the default assumption is that the production code is wrong.
- This includes loosening assertions, changing expected values to match observed values, removing cases, adding `if` guards that short-circuit the assertion, or commenting tests out.
- If you genuinely believe the test itself is wrong (it encodes the wrong behavior, has a typo, depends on something that legitimately changed, etc.), **STOP and ask the user.** Explain what the test currently asserts, what you think it should assert instead, and why. Wait for explicit permission before editing it.
- Mechanical, non-semantic edits to a test (renaming a symbol the test references, updating an import path after a move) are fine — but if the change touches an assertion or expected value, ask first.

## Quality gate — run before declaring a phase ready to commit

Go code must clear the three-part bar before you report a phase finished: zero surviving **and** zero uncovered mutations (`mutate4go`), CRAP ≤ 5 on every changed function (`crap4go`), and no duplication (`dry4go`).

Follow the `verify` skill for the full procedure. The essentials:

- Run it once per **phase**, after the last checklist item — not after every item. Mutation testing runs the test suite once per mutation and is far too slow to run mid-phase.
- **All three tools exit `0` even when they find problems.** Parse their output. Never infer a pass from exit status.
- Fix what fails and re-run before reporting. The "tests are the spec" rule above governs every fix — kill a surviving mutation by strengthening a test, never by weakening one.
- If a survivor looks like an equivalent mutant (no possible test can kill it), stop and ask the user rather than contorting a test around it.

### Strip mutate4go manifests before reporting the phase ready

`mutate4go` appends a manifest comment to every source file it processes:

```go
// mutate4go-manifest-begin
// {"version":1,"tested_at":"...","module_hash":"...","functions":[...]}
// mutate4go-manifest-end
```

**Leave them in place while you work.** The manifest is what lets `mutate4go` re-test only what changed, so it stays useful across every fix-and-re-run cycle. Don't strip it after one green file, or one green run.

**But it must not reach the commit.** Remove it only once you are certain the phase is actually done — every changed file mutated with `Survived: 0` and `Uncovered: 0`, every changed function at CRAP ≤ 5, `dry4go` clean, and any equivalent mutants settled with the user. If any of that is still open, keep the manifests and keep working.

When it all holds, strip them from every file you mutated:

```bash
sed -i '' '/^\/\/ mutate4go-manifest-begin$/,/^\/\/ mutate4go-manifest-end$/d' path/to/file.go
gofmt -w path/to/file.go
```

`gofmt` clears the blank line the manifest leaves behind, restoring the file byte-for-byte. On Linux, use `sed -i` without the `''`.

Then confirm the tree is clean — `go build ./...` and the ordinary test suite — and check `git diff` to be sure no manifest survived anywhere. Removing them costs only `mutate4go`'s differential re-run optimization, which the gate doesn't rely on: it runs `--mutate-all` regardless.

Report the gate results alongside the phase summary, with the numbers. If a criterion is unmet and you couldn't resolve it, say so explicitly — do not let it pass silently into the commit.

Skip the gate for phases that touched no Go code, and say that's why.

## Writing style: ASD-STE100 (Simplified Technical English)

You write prose during execution: commit messages, phase reports, code comments, and any text you add to the proposal. Write all of it according to the principles of ASD-STE100:

- Use simple, common words, each with one clear meaning. Prefer the shortest word that works.
- Use the active voice and name the doer: "The parser rejects empty input," not "Empty input is rejected."
- Use the present tense wherever possible.
- Keep sentences short: at most 20 words for a list item or an instruction, at most 25 words for descriptive prose.
- Write one instruction per sentence, and one topic per sentence.
- Keep each paragraph to one topic and at most 6 sentences.
- Use articles ("a", "the") and demonstratives ("this", "these") — do not drop them telegraphically.
- Break up noun clusters of more than three nouns.
- Use the same term for the same thing throughout the document — no elegant variation.
- Use vertical lists in place of long, complex sentences.
- Avoid idioms, slang, and unnecessary jargon.

Exact technical names (types, functions, commands, file paths) are exempt from the vocabulary rules — always write them precisely. Quoted material is exempt too: reproduce an error message, a log line, or a code comment verbatim.

Two places where this matters most:

- **Commit messages.** Name the doer and say what the commit does, in the present tense: "Extract FooParser from Handler," not "FooParser was extracted." Keep the subject line under 20 words.
- **Code comments.** You already write few of them. Make each one a single short sentence that says why the code is not obvious.

## When you finish

Report which checklist items were completed, note any deviations from the proposal (and why), and flag anything that should be cleaned up or followed up on.

If the proposal has been pushed to ClickUp (its `<article>` carries a `data-clickup-parent` attribute), remind the user they can run `/clickup-progress` to mirror the newly-checked steps and advance task status in ClickUp. Don't call any ClickUp tools from this skill — execution stays offline.
