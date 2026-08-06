---
name: clickup-sync
description: Push a finalized proposal into ClickUp — create one subtask per phase under a top-level task, attach the proposal file, and write the ClickUp task IDs back into the proposal. Idempotent, so re-running reconciles an edited proposal instead of duplicating tasks.
---

The user has a finalized proposal (an HTML document produced by `/propose`) and wants it projected into ClickUp: one subtask per phase under a given top-level task, with the proposal attached to the parent. This skill is the bridge from the proposal to ClickUp's task structure.

**The proposal HTML is the source of truth. ClickUp is a projection of it.** Every run reconciles the current proposal against ClickUp. The linkage that makes this work lives in the HTML: each phase carries a `data-clickup-task-id`, and the `<article>` carries `data-clickup-parent` / `data-clickup-list`. Because of those attributes the skill is idempotent — a phase that already has an ID is **updated**, a phase without one is **created** and gets its ID written back. This is also the re-sync path after the proposal changes: edit the proposal, re-run this skill, and existing subtasks update in place.

## Step 1 — Locate the proposal

Find the proposal in `<git-repo-root>/doc/work-sessions/<yyyy>/` (use `git rev-parse --show-toplevel`). If there's more than one and the target isn't obvious, ask the user which one. Read the whole file — you need the Approach section for phase goals and every `<details class="phase">` block.

## Step 2 — Resolve the parent task

Ask the user for the top-level ClickUp task URL if they haven't given it. Extract the task ID from the URL — it's the segment after `/t/` (e.g. `https://app.clickup.com/t/86abc123` → `86abc123`; custom IDs like `DEV-1234` are also valid).

Call `clickup_get_task` on that ID with `expand_statuses: true`. From the result capture:
- the **`list_id`** the task lives in — subtasks must be created in the same list,
- the task's existence/name (confirm with the user this is the right parent),
- the **available statuses** (for reference; status changes are `clickup-progress`'s job, not this skill's).

If the `<article>` already has `data-clickup-parent`, confirm it matches the URL the user gave; if they differ, ask before proceeding (the proposal may have been synced to a different task before).

## Step 3 — Parse the phases

From the HTML, for each `<details class="phase">` extract:
- **Phase title** — the `<span class="ph-title">` text (e.g. "Phase 2 — handle empty input").
- **Kind** — the `data-kind` attribute (`structural` / `behavioral`).
- **Steps** — each `<li>` in the `<ul class="checklist">`: the `<span>` text is the step; note any `<code>`/`<pre>` content (commands, snippets) attached to it.
- **Existing task ID** — the `data-clickup-task-id` attribute if present.

Derive a **1–2 sentence goal** for each phase from the phase's steps plus the proposal's Approach/Background sections — what this phase accomplishes and why, in plain language. The proposal template has no dedicated per-phase prose field, so you synthesize this; keep it honest and specific, not a restatement of the title.

## Step 4 — Build each subtask description

Each subtask's `markdown_description` must give a developer everything needed to pick up the phase cold. Use this shape:

```markdown
**Goal:** <1–2 sentence plain-language goal of the phase.>

_Kind: <Structural|Behavioral> · Phase of [<proposal filename>] (attached to the parent task)._

### Checklist
- [ ] <terse one-line action for step 1>
- [ ] <terse one-line action for step 2>
...

### Details
1. <fuller text for step 1 — the reason, the exact command in `code`, the file path, any snippet in a fenced block>
2. <fuller text for step 2>
...
```

- **Checklist** is concise: just the action per step, one line, so it's scannable and ClickUp renders it as live checkboxes. Mirror the proposal's steps 1:1 in order.
- **Details** carries the substance the checklist omits — reasons, exact commands, file paths, code/pseudocode in fenced blocks. If a step is fully self-explanatory in one line, its detail entry can just repeat it; don't pad.
- Reflect each step's **current checked state** from the HTML into the `- [ ]` / `- [x]` of the checklist (a re-sync of a partially-done proposal should show the right boxes). Do **not** set task status here — that's `clickup-progress`.

## Step 5 — Create or update each subtask

For each phase, in proposal order:

- **Has `data-clickup-task-id`** → `clickup_update_task` with that `task_id` and the new `markdown_description` (and `name` if the title changed). This is the reconcile path.
- **No ID** → `clickup_create_task` with `name` = phase title, `list_id` = the parent's list, `parent` = the parent task ID, and the `markdown_description`. Take the new task ID from the result and **immediately write it back** into the HTML with `Edit`: add `data-clickup-task-id="<id>"` to that phase's `<details ...>` tag.

Run these sequentially so each write-back Edit lands before the next phase (the Edits target distinct lines, but ordering keeps the file coherent if you re-read).

After the first successful create (or on first run), also write `data-clickup-parent="<parent-id>"` and `data-clickup-list="<list-id>"` onto the `<article>` tag if not already present, so `clickup-progress` and future re-syncs can find the linkage offline.

## Step 6 — Attach the proposal to the parent

Upload the proposal HTML as a file attachment on the **parent** task (not the subtasks):

1. Base64-encode the file: `base64 -i <proposal-path>` (macOS).
2. Call `clickup_attach_task_file` with `task_id` = parent ID, `file_data` = the base64 string, `file_name` = the proposal's basename (keep the `.html` extension).

Re-running attaches a fresh copy — ClickUp keeps multiple attachments. On a re-sync, ask the user whether they want a refreshed attachment or to skip it, so the parent doesn't accumulate stale copies.

## Step 7 — Report

Tell the user, per phase: created vs. updated, the subtask name, and its ClickUp ID. Confirm the proposal was attached. Note that the IDs are now stored in the proposal HTML and that `/clickup-progress` will push checkbox/status updates as they work.

## Hard constraints

- **Never create a duplicate subtask for a phase that already has a `data-clickup-task-id`.** If an ID is present but `clickup_get_task` says it's gone (deleted in ClickUp), tell the user and ask whether to recreate it before clearing the stale ID.
- Don't change task **status** in this skill — only structure and descriptions. Status is `clickup-progress`'s responsibility.
- Don't write production or test code; this is a sync tool.
- Preserve the proposal's checklist contract (`<li><input type="checkbox"> <span>…</span></li>`) — your write-backs only touch the `<details>`/`<article>` tags' attributes, never the checkboxes.
