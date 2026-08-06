---
name: clickup-progress
description: Push execution progress from a proposal into its ClickUp subtasks — mirror each phase's checked steps into the subtask description and advance status (inferred from checkbox state, confirmed before writing). Run it whenever you want to surface progress; execution itself stays offline.
---

The user has been working a proposal (via `/execute` or by hand), which checks off steps in the proposal HTML. This skill reflects that progress into the ClickUp subtasks `/clickup-sync` created — without coupling execution to ClickUp. You run it manually whenever you want ClickUp to catch up.

**The proposal HTML is the source of truth.** This skill reads the checkbox states and the stored task IDs out of the HTML and updates ClickUp to match. It never edits the proposal.

## Step 1 — Locate the proposal and the linkage

Find the proposal in `<git-repo-root>/doc/work-sessions/<yyyy>/` (use `git rev-parse --show-toplevel`); ask the user if ambiguous. Read it.

From the `<article>` tag read `data-clickup-parent` and `data-clickup-list`. If they're missing, the proposal was never synced — tell the user to run `/clickup-sync` first and stop.

## Step 2 — Read per-phase state

For each `<details class="phase">`:
- Read its `data-clickup-task-id`. If a phase has no ID, note it (it was added after the last sync) and tell the user it needs `/clickup-sync`; skip it here.
- Count its checklist `<input type="checkbox">`: total, and how many have the `checked` attribute.
- Capture each step's text and checked state so you can re-render the subtask's markdown checklist.

## Step 3 — Infer status

Discover the list's valid status names: call `clickup_get_task` on the parent (or any one subtask) with `expand_statuses: true`. Status names are workspace-specific, so map intent to whatever the list actually offers:

- **none checked** → the list's initial/open status (e.g. "to do", "open", "backlog").
- **some checked** → an in-progress status (e.g. "in progress", "in development").
- **all checked** → a done/closed status (e.g. "complete", "done", "closed").

Pick the closest available name for each. If the list has an ambiguous or unusual status set, surface the options to the user rather than guessing.

## Step 4 — Show the plan and get confirmation

Before writing anything, print a table the user can check: one row per phase with `phase title · checked X/Y · current status → proposed status · description will update (yes/no)`. Get the current status from `clickup_get_task` per subtask (or a `clickup_filter_tasks` over the parent's subtasks).

**Wait for explicit confirmation.** The user may correct a status mapping or tell you to skip a phase. Apply their corrections, then proceed only on a clear go-ahead.

## Step 5 — Apply

For each confirmed phase, `clickup_update_task` with:
- `task_id` = the phase's `data-clickup-task-id`,
- `status` = the confirmed status (omit if unchanged),
- `markdown_description` = the subtask description with its `### Checklist` re-rendered so each step's `- [ ]` / `- [x]` matches the proposal's checkbox state. Preserve the `**Goal:**` line and `### Details` section as they are — only the checklist boxes change.

To rebuild the description cleanly, fetch the current description with `clickup_get_task` (`include: ["description"]`), swap only the checklist box states, and write it back. If reconstructing it is unreliable, regenerate the full description the same way `/clickup-sync` does (goal + checklist + details) so nothing is lost.

## Step 6 — Report

Per phase: old status → new status and how many boxes are now checked in ClickUp. Note any phases skipped (no ID yet, or user-excluded) and remind the user to `/clickup-sync` if any phase lacked an ID.

## Hard constraints

- **Never edit the proposal HTML** — this skill only reads it and writes to ClickUp.
- **Never write status (or anything) without showing the plan and getting confirmation first.**
- Don't touch a phase that has no `data-clickup-task-id`; route it to `/clickup-sync` instead of creating tasks here.
- Don't write production or test code.
