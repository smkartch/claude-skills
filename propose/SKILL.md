---
name: propose
description: Phase 2 of the agentic coding workflow. Drafts a proposal document, runs it through parallel critic agents to push back on approach and scope, then iterates with the user until approved. No code is written.
---

The user wants to plan a coding task. Your job is to produce a proposal document, sharpen it with a critic pass from multiple agents, and iterate with the user until they approve it. Do **not** write any production or test code until the user explicitly says to proceed.

## Step 1 — Read the research

Locate the research document in `<git-repo-root>/doc/work-sessions/<yyyy>/` (use `git rev-parse --show-toplevel`). If one exists for this topic, read it in full before drafting. If none exists, ask the user whether they want to run `/research` first or proceed without it.

## Step 2 — Draft the proposal

Write the draft as a single self-contained HTML document at:

```
<git-repo-root>/doc/work-sessions/<yyyy>/<yyyy-mm-dd_hh-mm-ss>-proposal-<terse-description>.html
```

- `<yyyy-mm-dd_hh-mm-ss>` — from `date '+%Y-%m-%d_%H-%M-%S'`
- `<terse-description>` — short hyphen-separated topic slug, ideally matching the research doc.

### Document structure

Emit a complete, standalone HTML5 document — all CSS and JS inline, no external resources — so it renders beautifully when opened directly in a browser, online or off. This is a document someone has to *read and approve*; make it a pleasure to read. Aim for the feel of a well-crafted editorial blog — the kind a thoughtful designer would publish: confident serif headings, clean readable body text, generous whitespace, and one restrained accent. Calm and refined, never developer-boilerplate or app-dashboard. Use this template as your starting point and adapt it to the topic:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Proposal: <Topic></title>
<style>
  :root {
    --paper: #fcfbf7; --surface: #ffffff; --ink: #23201d; --soft: #57514a; --muted: #938b80;
    --accent: #a8324a; --line: #e8e3d8; --rule: #d9d3c6;
    --serif: "New York", ui-serif, Georgia, "Iowan Old Style", "Palatino Linotype", "Times New Roman", serif;
    --sans: -apple-system, BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, system-ui, sans-serif;
    --mono: ui-monospace, "SF Mono", "JetBrains Mono", Menlo, monospace;
  }
  /* Dark palette — automatic, or forced via the toggle (data-theme on <html>) */
  @media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) {
      --paper: #1a1817; --surface: #221f1c; --ink: #ece7dd; --soft: #c0b8ac; --muted: #8d857a;
      --accent: #e58aa0; --line: #322e29; --rule: #3b3631;
    }
  }
  :root[data-theme="dark"] {
    --paper: #1a1817; --surface: #221f1c; --ink: #ece7dd; --soft: #c0b8ac; --muted: #8d857a;
    --accent: #e58aa0; --line: #322e29; --rule: #3b3631;
  }
  * { box-sizing: border-box; }
  :focus-visible { outline: 2px solid var(--accent); outline-offset: 3px; border-radius: 2px; }
  html { -webkit-font-smoothing: antialiased; text-rendering: optimizeLegibility; }
  body {
    font-family: var(--sans); font-size: 1.16rem; line-height: 1.75;
    color: var(--ink); background: var(--paper);
    max-width: 41rem; margin: 0 auto; padding: 0 1.4rem 6rem;
    animation: fade .6s ease both;
  }
  @keyframes fade { from { opacity: 0; } to { opacity: 1; } }
  @media (prefers-reduced-motion: reduce) { *, ::before, ::after { animation: none !important; transition: none !important; } }

  /* Masthead */
  .masthead { margin: 4.5rem 0 3rem; }
  .kicker { font-family: var(--sans); text-transform: uppercase; letter-spacing: .22em; font-size: .72rem; font-weight: 600; color: var(--accent); margin: 0 0 1rem; }
  h1 { font-family: var(--serif); font-weight: 600; font-size: clamp(2.3rem, 6vw, 3.2rem); line-height: 1.08; letter-spacing: -.01em; margin: 0; color: var(--ink); }
  .dek { font-family: var(--serif); font-style: italic; font-size: 1.4rem; line-height: 1.45; color: var(--soft); margin: 1.1rem 0 0; }
  .byline { font-size: .82rem; color: var(--muted); margin: 1.6rem 0 0; padding-top: 1.4rem; border-top: 1px solid var(--rule); letter-spacing: .02em; }

  /* Section headings */
  h2 { font-family: var(--serif); font-weight: 600; font-size: 1.9rem; line-height: 1.2; letter-spacing: -.01em; margin: 3.4rem 0 1rem; color: var(--ink); }
  h3 { font-family: var(--serif); font-weight: 600; font-size: 1.3rem; margin: 2rem 0 .6rem; }
  p { margin: 1.1rem 0; }
  a { color: var(--accent); text-underline-offset: .15em; text-decoration-thickness: 1px; }

  /* Drop cap on the opening paragraph */
  .lede-para::first-letter { font-family: var(--serif); font-weight: 600; float: left; font-size: 3.6rem; line-height: .72; padding: .35rem .55rem 0 0; color: var(--accent); }

  ul, ol { padding-left: 1.3rem; }
  li { margin: .5rem 0; }

  pre { font-family: var(--mono); background: var(--surface); border: 1px solid var(--line); padding: 1.1rem 1.2rem; overflow-x: auto; border-radius: 6px; font-size: .92rem; line-height: 1.6; margin: 1.6rem 0; }
  code { font-family: var(--mono); font-size: .9em; background: rgba(168,50,74,.08); padding: .1rem .35rem; border-radius: 4px; }
  pre code { padding: 0; background: none; }

  /* Pull-quote lede (TL;DR) */
  .summary { border-left: 3px solid var(--accent); padding: .2rem 0 .2rem 1.4rem; margin: 2rem 0; }
  .summary .label { display: block; font-size: .72rem; text-transform: uppercase; letter-spacing: .18em; color: var(--muted); margin-bottom: .4rem; }
  .summary p { font-family: var(--serif); font-style: italic; font-size: 1.35rem; line-height: 1.5; color: var(--soft); margin: 0; }

  /* Aside (risks) */
  .aside { background: var(--surface); border: 1px solid var(--line); border-radius: 8px; padding: 1.1rem 1.3rem; margin: 1.8rem 0; }
  .aside .label { display: block; font-size: .72rem; text-transform: uppercase; letter-spacing: .16em; color: var(--accent); font-weight: 600; margin-bottom: .4rem; }
  .aside p { margin: 0; color: var(--soft); }

  /* Checklist progress — quiet, editorial */
  .meter { margin: 1.4rem 0 2rem; }
  .meter-track { height: 2px; background: var(--rule); overflow: hidden; }
  .meter-fill { height: 100%; width: 0; background: var(--accent); transition: width .5s ease; }
  .meter-label { font-size: .74rem; text-transform: uppercase; letter-spacing: .16em; color: var(--muted); margin-top: .6rem; font-variant-numeric: tabular-nums; }

  /* Phases as elegant sections, not cards */
  .phase { border-top: 1px solid var(--rule); margin: 0; }
  .phase > summary { cursor: pointer; list-style: none; display: flex; align-items: baseline; justify-content: space-between; gap: 1rem; padding: 1.1rem 0 .7rem; }
  .phase > summary::-webkit-details-marker { display: none; }
  .phase .ph-title { font-family: var(--serif); font-weight: 600; font-size: 1.25rem; color: var(--ink); }
  .phase .ph-kind { font-size: .68rem; text-transform: uppercase; letter-spacing: .16em; color: var(--muted); white-space: nowrap; }
  .phase > summary::after { content: "+"; color: var(--muted); font-size: 1.1rem; margin-left: auto; transition: transform .2s; }
  .phase[open] > summary::after { content: "\2013"; }

  ul.checklist { list-style: none; padding-left: 0; margin: .2rem 0 1.4rem; }
  ul.checklist li { display: flex; align-items: flex-start; gap: .7rem; margin: .55rem 0; font-size: 1.02rem; line-height: 1.55; }
  ul.checklist input[type="checkbox"] { appearance: none; -webkit-appearance: none; margin: .35rem 0 0; width: 1.05rem; height: 1.05rem; flex: none; border: 1.5px solid var(--muted); border-radius: 4px; cursor: pointer; position: relative; transition: background .15s, border-color .15s; }
  ul.checklist input[type="checkbox"]:checked { background: var(--accent); border-color: var(--accent); }
  ul.checklist input[type="checkbox"]:checked::after { content: ""; position: absolute; left: .3rem; top: .12rem; width: .28rem; height: .55rem; border: solid #fff; border-width: 0 2px 2px 0; transform: rotate(45deg); }
  ul.checklist input[type="checkbox"]:checked + span { color: var(--muted); text-decoration: line-through; text-decoration-color: var(--rule); }

  footer { margin-top: 4rem; padding-top: 1.6rem; border-top: 1px solid var(--rule); color: var(--muted); font-size: .82rem; font-style: italic; }

  /* Interactive controls — quiet, editorial */
  button { font-family: var(--sans); cursor: pointer; }
  .toolbar { display: flex; justify-content: flex-end; margin: 1.6rem 0 -1.4rem; }
  .btn { font-size: .68rem; text-transform: uppercase; letter-spacing: .15em; color: var(--soft); background: transparent; border: 1px solid var(--rule); border-radius: 99px; padding: .42rem .95rem; transition: border-color .15s, color .15s; }
  .btn:hover { color: var(--accent); border-color: var(--accent); }
  .ck-tools { display: flex; gap: 1.4rem; margin: -.4rem 0 1.6rem; }
  .btn-link { background: none; border: 0; padding: 0 0 1px; font-size: .68rem; text-transform: uppercase; letter-spacing: .15em; color: var(--muted); border-bottom: 1px solid transparent; transition: color .15s, border-color .15s; }
  .btn-link:hover { color: var(--accent); border-bottom-color: var(--accent); }
  .code-wrap { position: relative; }
  .code-wrap .copy { position: absolute; top: .55rem; right: .55rem; opacity: 0; font-family: var(--sans); font-size: .62rem; text-transform: uppercase; letter-spacing: .12em; padding: .28rem .6rem; border: 1px solid var(--rule); border-radius: 6px; background: var(--paper); color: var(--muted); transition: opacity .15s, color .15s, border-color .15s; }
  .code-wrap:hover .copy, .code-wrap .copy:focus-visible { opacity: 1; }
  .code-wrap .copy:hover, .code-wrap .copy.copied { color: var(--accent); border-color: var(--accent); }
</style>
</head>
<body>
<article>
<div class="toolbar">
  <button class="btn" id="theme-toggle" aria-pressed="false">Dark mode</button>
</div>
<header class="masthead">
  <p class="kicker">Proposal</p>
  <h1><Topic></h1>
  <p class="dek"><one honest, well-crafted sentence — the editorial dek that frames what this changes and why it matters></p>
  <p class="byline">Drafted <yyyy-mm-dd> &nbsp;·&nbsp; Plan-before-code workflow &nbsp;·&nbsp; <N> min read</p>
</header>

<section>
  <h2>Background</h2>
  <p class="lede-para">Why this work is needed; relevant context. The first paragraph gets a drop cap, so open with a real sentence, not a heading. Reference the research document if one exists.</p>
  <div class="summary">
    <span class="label">In short</span>
    <p><one- or two-sentence distillation of the whole proposal></p>
  </div>
</section>

<section>
  <h2>Approach</h2>
  <p>Detailed description of the chosen approach.</p>
  <ul>
    <li>Key design decisions and why they were made</li>
    <li>Alternatives considered and why they were rejected</li>
    <li>Relevant code snippets, interfaces, or pseudocode (in <code>&lt;pre&gt;&lt;code&gt;</code> blocks)</li>
    <li>File paths that will be created or modified</li>
  </ul>
</section>

<section>
  <h2>Trade-offs &amp; Risks</h2>
  <div class="aside">
    <span class="label">Worth weighing</span>
    <p>Honest assessment of downsides, open questions, or unknowns.</p>
  </div>
</section>

<section>
  <h2>Implementation Checklist</h2>
  <div class="meter">
    <div class="meter-track"><div class="meter-fill" id="meter-fill"></div></div>
    <div class="meter-label" id="meter-label">0 of 0 steps complete</div>
  </div>
  <div class="ck-tools">
    <button class="btn-link" id="expand-all">Expand all</button>
    <button class="btn-link" id="collapse-all">Collapse all</button>
  </div>
  <!-- See requirements below -->
</section>

<footer><p>Drafted as part of the plan-before-code workflow — reviewed and approved before a line of code is written.</p></footer>
</article>

<script>
  (function () {
    // Checklist meter
    var boxes = Array.prototype.slice.call(document.querySelectorAll('ul.checklist input[type="checkbox"]'));
    var fill = document.getElementById('meter-fill');
    var label = document.getElementById('meter-label');
    function render() {
      var done = boxes.filter(function (b) { return b.checked; }).length;
      var pct = boxes.length ? Math.round((done / boxes.length) * 100) : 0;
      fill.style.width = pct + '%';
      label.textContent = boxes.length && done === boxes.length
        ? 'All ' + boxes.length + ' steps complete'
        : done + ' of ' + boxes.length + ' steps complete';
    }
    boxes.forEach(function (b) { b.addEventListener('change', render); });
    render();

    // Theme toggle (overrides prefers-color-scheme for this view)
    var root = document.documentElement, toggle = document.getElementById('theme-toggle');
    function isDark() {
      var set = root.getAttribute('data-theme');
      if (set) return set === 'dark';
      return window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
    }
    function syncToggle() {
      var dark = isDark();
      toggle.textContent = dark ? 'Light mode' : 'Dark mode';
      toggle.setAttribute('aria-pressed', String(dark));
    }
    toggle.addEventListener('click', function () {
      root.setAttribute('data-theme', isDark() ? 'light' : 'dark');
      syncToggle();
    });
    syncToggle();

    // Expand / collapse all phases
    var phases = Array.prototype.slice.call(document.querySelectorAll('details.phase'));
    var ea = document.getElementById('expand-all'), ca = document.getElementById('collapse-all');
    if (ea) ea.addEventListener('click', function () { phases.forEach(function (p) { p.open = true; }); });
    if (ca) ca.addEventListener('click', function () { phases.forEach(function (p) { p.open = false; }); });

    // Copy buttons on code blocks
    Array.prototype.slice.call(document.querySelectorAll('pre')).forEach(function (pre) {
      var wrap = document.createElement('div');
      wrap.className = 'code-wrap';
      pre.parentNode.insertBefore(wrap, pre);
      wrap.appendChild(pre);
      var btn = document.createElement('button');
      btn.className = 'copy'; btn.type = 'button'; btn.textContent = 'Copy';
      btn.setAttribute('aria-label', 'Copy code to clipboard');
      btn.addEventListener('click', function () {
        var text = pre.innerText;
        var done = function () { btn.textContent = 'Copied'; btn.classList.add('copied'); setTimeout(function () { btn.textContent = 'Copy'; btn.classList.remove('copied'); }, 1500); };
        if (navigator.clipboard && navigator.clipboard.writeText) { navigator.clipboard.writeText(text).then(done, done); }
        else { var t = document.createElement('textarea'); t.value = text; document.body.appendChild(t); t.select(); try { document.execCommand('copy'); } catch (e) {} document.body.removeChild(t); done(); }
      });
      wrap.appendChild(btn);
    });
  })();
</script>
</body>
</html>
```

Notes on the template:
- **Keep the structural contract intact.** Each checklist item must stay an `<li><input type="checkbox"> <span>...text...</span></li>` so the `execute` skill can mark it done by adding the `checked` attribute. The meter reads these on load, so a proposal that comes back from `execute` half-checked shows the right fill automatically.
- Wrap each phase in `<details class="phase" data-kind="structural|behavioral">` with a `<summary>` (see the checklist template below). `data-kind` is there if you want to style the two kinds differently; the visible label lives in `.ph-kind`.
- **Leave room for the ClickUp linkage.** Don't add it yourself, but be aware the `/clickup-sync` skill writes a `data-clickup-task-id` attribute onto each `<details class="phase">` and `data-clickup-parent` / `data-clickup-list` onto the `<article>` when the proposal is pushed to ClickUp. These attributes are how phases stay mapped to their ClickUp subtasks across re-syncs. Never strip them when revising a proposal — they carry the linkage that keeps sync idempotent.
- Escape literal `<`, `>`, and `&` as `&lt;`, `&gt;`, and `&amp;` in prose and code. Put code, interfaces, and pseudocode in `<pre><code>...</code></pre>`.
- Inline everything. No web fonts, CDNs, or external scripts — it must render fully offline. The serif headings rely on the system serif (New York on Apple, Georgia elsewhere); don't reach for a web font.

### Voice & polish

The aesthetic is **a well-made editorial blog** — the kind a thoughtful designer would publish. Serif display headings, a clean sans for reading, generous whitespace, a warm paper background, one restrained accent. Calm, confident, and easy on the eyes. Not an app dashboard, not a corporate spec, not cutesy. Within that, give the proposal a voice:
- **Write like a good essayist, not a ticket.** Lead with a strong dek that frames the stakes. Full sentences, clear paragraphs, a point of view. The reader should want to keep reading — earn that with prose, not decoration.
- **Let typography do the work.** Hierarchy, rhythm, and whitespace carry the design. Resist emoji, badges, and color blocks; a single well-placed pull-quote or drop cap does more than a dozen ornaments.
- **One accent, used sparingly.** The accent color is for links, the drop cap, small rules, and the checked state — not large fills. Keep the palette warm and quiet.
- **Substance first.** The meter, collapsible phases, refined checkboxes, and the controls below serve a genuinely useful document; they're never the point. If a flourish competes with the writing, cut it.
- **Interactive controls, kept quiet.** The template ships working, accessible controls: a dark/light **theme toggle**, **Expand all / Collapse all** for the phases, **copy buttons** on every code block, and the live checklist meter. They're styled as understated text/pill buttons in the accent color — never loud. Keep these wired up; if a topic calls for another control (e.g. a filter or a jump-to-phase nav), add it in the same restrained style and make sure it's keyboard-accessible with a sensible `aria` label.
- **Respect the reader.** Comfortable measure (~40rem), real line-height, high contrast. The template's dark mode is a warm ink (not stark black); honor `prefers-reduced-motion`.
- You may retune the accent hue, the dek, and the closing line to fit the topic — but keep the editorial restraint, the legibility, and the checklist contract unchanged.

### Writing style: ASD-STE100 (Simplified Technical English)

Write all prose in the proposal according to the principles of ASD-STE100:

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

STE and the editorial voice above are not in tension. STE governs the sentences; the voice governs the structure, the typography, and the point of view. A dek, a drop cap, and a strong opening paragraph all survive the rules — short, plain, active sentences still argue a position. Where the two pull against each other, clarity wins.

The proposal is a document someone reads once and then approves or rejects. Every sentence they have to read twice is a defect.

### Implementation Checklist requirements

Each item must be a concrete, verifiable step, written as one instruction in at most 20 words. **Organize using a red/green TDD cycle wherever applicable:**

1. Write or modify a test that captures the desired behavior
2. Run the test — confirm it fails for the right reason (compile error is acceptable if production code doesn't exist yet)
3. Write or modify production code to make the test pass
4. Run the tests again — confirm they pass
5. Run coverage on the changed files — confirm 100% line and branch coverage of the production code added or modified in this phase

#### 100% coverage of changed code

Every behavioral phase must end with **100% line and branch coverage of the production code that phase adds or modifies.** This is non-negotiable — if a line or branch can't be reached by a test, either the code is dead (delete it) or the test is missing (add it). The coverage check belongs inside the phase, not as a sweep at the end, so gaps land in the same commit as the code that caused them.

When drafting the checklist:
- Pick a coverage tool appropriate to the language (e.g. `go test -cover`, `pytest --cov`, `cargo tarpaulin`, `c8`/`nyc`) and name it explicitly in the relevant steps.
- Scope the coverage check to the files touched in that phase, not the whole repo — repo-wide coverage is a separate concern.
- If a line is genuinely untestable (e.g. a defensive branch the type system already rules out), call it out in `Trade-offs & Risks` with the justification rather than silently excluding it.
- Structural phases don't need a coverage step (no behavior changed), but tests must still pass before and after.

#### One commit per phase

**Every phase is its own commit.** When you write the checklist, plan phases as commit-sized units of work — small enough that each one is reviewable on its own, scoped so the working tree is green at the end. The execute step will pause at each phase boundary and wait for the user to commit before moving on.

#### Structural OR behavioral, never both

Each phase must be **either** a structural change **or** a behavioral change — never both in the same phase, and therefore never both in the same commit.

- **Structural changes** rearrange code without changing what it does: renames, moves, extractions, inlines, reformatting, splitting/merging files, introducing an interface that nothing yet uses differently. Tests should pass before and after with no assertion changes.
- **Behavioral changes** change what the code does: new behavior, bug fixes, changed outputs, new branches. These are the phases that follow the red/green TDD cycle.

If a planned phase mixes both, split it into two phases (and therefore two commits). Name each phase so the kind is obvious — e.g., `Phase 1 (structural): extract FooParser`, `Phase 2 (behavioral): handle empty input`.

#### Order: structural phases first, then behavioral

Where possible, **front-load all structural phases before any behavioral phases.** The intent is that the user can land and deploy the structural changes as a batch — verify nothing broke in production — and then land and deploy the behavioral changes as a separate batch. Two clean deploys instead of an interleaved one.

If a behavioral phase genuinely needs a structural change that no other behavioral phase needs, that structural phase can immediately precede it rather than going at the front. Call this out in the phase description so the reasoning is visible.

Group steps into named phases. Render each phase as a collapsible `<details class="phase" data-kind="structural|behavioral">`: a `<summary>` containing a `<span class="ph-title">` with the phase name and a `<span class="ph-kind">` with the kind (Structural / Behavioral), then a `<ul class="checklist">` with one step per `<li>`. Wrap each step's text in a `<span>` (the strike-through-on-check styling targets it) and keep the bare `<input type="checkbox">` so `execute` can mark a step done by adding the `checked` attribute. Open the first phase (`<details ... open>`) so the reader lands on something. The phases go inside the Implementation Checklist `<section>`, after the `.meter`:

```html
<details class="phase" data-kind="structural" open>
  <summary><span class="ph-title">Phase 1 — <name></span><span class="ph-kind">Structural</span></summary>
  <ul class="checklist">
    <li><input type="checkbox"> <span><refactor step></span></li>
    <li><input type="checkbox"> <span>Run tests, confirm still passing (no assertion changes)</span></li>
  </ul>
</details>

<details class="phase" data-kind="behavioral">
  <summary><span class="ph-title">Phase 2 — <name></span><span class="ph-kind">Behavioral</span></summary>
  <ul class="checklist">
    <li><input type="checkbox"> <span>Write test for <behavior> — expect failure: <reason></span></li>
    <li><input type="checkbox"> <span>Run tests, confirm failure</span></li>
    <li><input type="checkbox"> <span>Implement <behavior></span></li>
    <li><input type="checkbox"> <span>Run tests, confirm passing</span></li>
    <li><input type="checkbox"> <span>Run <code><coverage command></code> on <changed files>, confirm 100% line and branch coverage</span></li>
  </ul>
</details>
```

## Step 3 — Critic pass

Send a single message with two `Agent` tool calls (run them in parallel). Each critic gets the proposal path and a focused lens:

- **Technical soundness critic** — does the approach actually solve the problem? Are there design flaws, race conditions, missing error paths, or interfaces that don't fit the existing codebase? Are the alternatives genuinely considered or strawmanned? Does every behavioral phase include a coverage step that confirms 100% line and branch coverage of the code it changes, with a concrete coverage command named?
- **Scope & simplicity critic** — is the proposal larger than it needs to be? Are there abstractions designed for hypothetical futures? Is the checklist over-engineered or missing concrete steps? Could a smaller change get 80% of the value?

Brief each critic to:
- Read the full proposal at the given path.
- Read the research document if one exists.
- Return a prioritized punch list under 400 words. Each item: what's wrong, where in the proposal, and what would be better.
- Push back hard. The proposal is a draft — they're not being polite, they're stress-testing it.
- Flag prose that breaks the ASD-STE100 rules: sentences over 25 words, passive voice with no named doer, inconsistent terms for one thing, noun stacks, checklist items over 20 words.

Use `subagent_type: "general-purpose"` for both.

## Step 4 — Revise

Apply valid critiques. Write every revision in ASD-STE100, the same as the draft. If two critics disagree, exercise judgment — prefer the one with more specific evidence. If a critique would meaningfully change the approach, briefly note it in `Trade-offs & Risks` so the user can see what was considered.

One critic round is enough before showing the user. Do not loop — the user is the next reviewer.

## Step 5 — Hand off to the user

Tell the user where the proposal was written, summarize the critic findings you incorporated (and any you rejected, briefly), and ask them to review.

## Step 6 — Iterate

Write all new or revised prose in ASD-STE100, the same as the draft.

**In-session feedback** — if the user types corrections in chat, update the proposal document, do **not** start implementing, and repeat until they explicitly approve.

**Out-of-session feedback** — the user can edit the proposal file directly, marking each note with an `UPPERCASE_WORD:` annotation (e.g. `PROBLEM:`, `CORRECTION:`, `QUESTION:`, `TODO:`). When they return, re-invoke `/propose` on the same document: read all annotations, resolve any ambiguous ones through a clarification loop, rewrite the proposal in place, and return to a wait-for-approval state.

## Optional — push to ClickUp

Once the proposal is approved (or whenever it's been edited), the user can run `/clickup-sync` to project it into ClickUp: one subtask per phase under a top-level task, the proposal attached to the parent, and the ClickUp task IDs written back into this document. It's idempotent, so re-running after an edit reconciles the existing subtasks instead of duplicating them. ClickUp is entirely optional — the proposal workflow never requires it. Mention this once when handing off; don't call any ClickUp tools from this skill.

## Hard constraints

- No production or test code at any point in this skill.
- Do not start implementation even if the user says the proposal looks good in passing — wait for unambiguous approval or an explicit `/execute` invocation.
- Each subagent prompt must be fully self-contained.
