---
name: verify
description: Verify Go code against the three-part quality bar — no surviving or uncovered mutations (mutate4go), CRAP score of 5 or less (crap4go), and no duplication (dry4go). Run after writing or changing Go code, as the final gate of /execute, or on demand.
---

The user holds Go code to a three-part bar. Code you write must:

1. **Be well-tested** — evidenced not by coverage percentage but by `mutate4go` finding **zero surviving and zero uncovered mutations**.
2. **Have a CRAP score of five or less** — measured by `crap4go`.
3. **Be DRY** — no duplication reported by `dry4go`.

Your job is to run these tools against the code that changed, interpret the results, fix what fails, and re-run until the bar is met. Coverage percentage is not the goal and is not evidence of anything on its own.

## Critical: these tools always exit 0

**All three tools exit `0` whether or not they find problems.** A run with surviving mutations, a CRAP score of 90, and a dozen duplicate blocks still exits `0`.

Never gate on exit status. Never chain with `&&` and assume success. **You must read and parse the output text.** If you report a pass because a command "succeeded," you have reported a false pass.

## Step 0 — Preflight

Confirm the tools are installed:

```bash
command -v mutate4go crap4go dry4go
```

Any that are missing:

```bash
go install github.com/unclebob/mutate4go/cmd/mutate4go@latest
go install github.com/unclebob/crap4go/cmd/crap4go@latest
go install github.com/unclebob/dry4go/cmd/dry4go@latest
```

Both `crap4go` and `mutate4go` write coverage data to `target/coverage/coverage.out`. If `target/` is not in `.gitignore`, tell the user and offer to add it. Do not commit `target/`.

Determine what changed — the bar applies to code the user wrote or touched, not to the whole repo:

```bash
git diff --name-only HEAD -- '*.go'
git ls-files --others --exclude-standard -- '*.go'
```

Work from that file set. Exclude `_test.go` files from mutation targets — you mutate production code, not tests. If the user asked to verify something specific, scope to that instead.

## Step 1 — dry4go (fastest, run first)

```bash
dry4go .
```

Or scope it: `dry4go ./internal ./cmd`

**Clean output:** `No duplicate candidates found.`

**Failure output:**
```
DUPLICATE score=0.89
  internal/billing/invoice.go:12-25
  internal/billing/receipt.go:30-44
```

Any `DUPLICATE` block involving changed code is a failure. Extract the common logic. Do **not** raise `--threshold` to silence a report — that is gaming the metric, not fixing the duplication.

Duplication between two files you did not touch is pre-existing. Report it, don't fix it unsolicited.

## Step 2 — crap4go

Run from the module root:

```bash
crap4go
```

Filter to relevant paths when the repo is large: `crap4go internal/billing`

**Output:**
```
CRAP Report
===========
Function                       Package              CC    Cov%     CRAP
------------------------------------------------------------------------
Classify                       smoke                 3   40.0%      4.9
```

The formula is `CRAP = CC² × (1 − coverage)³ + CC`.

**The bar: every function you wrote or changed scores ≤ 5.0.** On an existing codebase the report will list pre-existing offenders — those are not your gate. Report them as observations; fix only what you touched.

Two ways to lower a CRAP score: reduce cyclomatic complexity, or increase real coverage. Prefer reducing complexity — a function simple enough not to need heroic testing is the actual goal.

**A passing CRAP score proves very little on its own.** In a verified example, a function at 40% coverage scored 4.9 — a pass — while two of its three branches were never executed. This is precisely why step 3 exists. Never report the bar as met on the strength of CRAP alone.

## Step 3 — mutate4go (slowest, run last)

`mutate4go` takes **one source file at a time**, and runs the test suite once per mutation. Loop over changed production files:

```bash
mutate4go path/to/file.go
```

Useful flags:
- `--max-workers N` — parallelize; use this, the tool is slow
- `--scan` — count mutation sites without running tests, for a cheap preview
- `--mutate-all` — run every covered mutation even when a manifest exists
- `--test-command CMD` — if the repo's tests need more than `go test ./...`

**Re-runs are differential by default.** Once a manifest exists, `mutate4go` only re-tests changed functions. For a final gate, run `--mutate-all` so the pass reflects the whole file.

**Output:**
```
Mutation Report
===============
Killed:    12
Survived:   1
Uncovered:  2

Survivors:
  line 4 > -> >= func/Classify
```

**Do not be alarmed by `FAIL` in the output.** While mutating, the tool prints the failing test run for every mutant it kills — a healthy run is full of `FAIL smoke 0.29s` and `--- FAIL: TestClassify` lines, each one a mutation your tests caught. `FAIL` here means *success*. Read only the `Mutation Report` block at the end for the verdict, and never report a failure on the strength of these lines.

**The bar: `Survived: 0` AND `Uncovered: 0`.**

- **Survived** — the mutation ran and no test noticed. A test asserts too weakly.
- **Uncovered** — the mutation site was never executed at all. No test reaches this code. Treat these as at least as serious as survivors; they are invisible to coverage-flavored intuition.

### `mutate4go` rewrites your source files

It appends a manifest comment to each file it processes:

```go
// mutate4go-manifest-begin
// {"version":1,"tested_at":"...","module_hash":"...","functions":[...]}
// mutate4go-manifest-end
```

This is expected, not corruption — it's what lets `mutate4go` re-test only changed functions instead of redoing the whole file on every pass.

**Leave it exactly where it is for now.** You will be looping — fixing tests and re-running — and the manifest is working for you the entire time. Do not remove it after a single green run, and do not remove it from one file because that file looks finished while others are still failing.

The manifest is build metadata with a timestamp inside and must not be committed, so it does get cleaned up — but only in Step 5, once every tool passes on every file and you are genuinely done mutating. Until that moment, treat it as part of the working state.

### Fixing survivors and uncovered sites

For each one, write or strengthen a test that fails against the mutant and passes against the original.

Your existing discipline applies with full force here: **tests are the spec.** Never weaken an assertion, delete a case, or narrow a test to make the report go green. The correct response to a survivor is always *more* assertion, never less.

Do not delete production code to eliminate a mutation site unless the code is genuinely dead — and say so explicitly if you do.

### Equivalent mutants

Some mutations produce semantically identical programs and **cannot be killed by any test**. A common shape is a boundary that is unreachable given the input domain, or a change inside a branch whose result is discarded.

When you believe a survivor is an equivalent mutant:

- **Stop. Do not contort a test to kill it**, and do not silently accept it.
- Report it to the user: the exact mutation, the function, and your reasoning for why no test can distinguish it.
- Let the user decide. Often the real fix is simplifying the code so the site stops existing.

A genuinely equivalent mutant is the one legitimate reason the bar may not reach zero. Everything else is a missing test.

## Step 4 — Loop

Fix, then re-run **the tool that failed** plus anything the fix could have disturbed:

- Added tests → re-run `mutate4go` on that file and `crap4go` (coverage moved).
- Extracted a duplicate → re-run `dry4go` and `crap4go` (complexity moved), and `mutate4go` on both touched files.
- Reduced complexity → re-run all three.

Keep going until clean. Run the full ordinary test suite at the end to confirm nothing regressed.

**Do not clean up manifests during this loop.** They are load-bearing until the loop exits.

## Step 5 — Clean up manifests (only once everything passes)

Do not begin this step until **all** of the following are true:

- Every changed production file has been mutated, with `Survived: 0` and `Uncovered: 0`.
- Every changed function is at CRAP ≤ 5.
- `dry4go` reports no duplicates in changed code.
- Any suspected equivalent mutants have been raised with the user and resolved.

If any of those is still open, you are not done mutating — go back to Step 4. A partially-verified file keeps its manifest.

Once all of it holds, strip the manifest from every file you mutated:

```bash
sed -i '' '/^\/\/ mutate4go-manifest-begin$/,/^\/\/ mutate4go-manifest-end$/d' path/to/file.go
gofmt -w path/to/file.go
```

`gofmt` removes the blank line the manifest leaves behind, restoring the file byte-for-byte. On Linux, drop the `''` after `-i`.

Then confirm you didn't break anything on the way out:

- `go build ./...`
- the ordinary test suite
- `git diff` — no `mutate4go-manifest` line survives anywhere

Removing the manifests costs only the differential re-run optimization, and only for a future run. If you later need to re-verify one of these files, just run `--mutate-all` again.

## Step 6 — Report

State each of the three criteria and whether it passed, with the numbers:

```
dry4go     PASS  no duplicate candidates
crap4go    PASS  highest changed function: ParseAddress, CC 4, 100.0%, CRAP 4.0
mutate4go  PASS  invoice.go: 23 killed, 0 survived, 0 uncovered
                 receipt.go: 11 killed, 0 survived, 0 uncovered
```

Rules for this report:

- **Be honest about partial passes.** If one criterion is unmet, say so plainly and show the numbers. A gate that reports success it didn't verify is worse than no gate.
- Name anything you skipped and why — a file not mutated, a package not scanned.
- List pre-existing violations you observed but did not fix, separately from your own.
- Flag any suspected equivalent mutants for the user's judgment.

Do not describe the bar as met unless you have seen `Survived: 0`, `Uncovered: 0`, every changed function at CRAP ≤ 5, no `DUPLICATE` blocks in changed code, and a `git diff` free of leftover `mutate4go` manifests.
