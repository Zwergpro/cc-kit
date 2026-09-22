---
name: code-flow-analysis
description: Trace an end-to-end operation through a codebase - from its entry point (API handler, CLI command, message consumer) through every async hop (state machines, job queues, workers, workflow engines) - and write it up as two Markdown documents sharing step IDs - a short "steps only" version and a detailed version with code references, DB writes, retries and pitfalls, both with ASCII flow graphs. Use this whenever the user asks to analyze, trace, map, document or explain "the flow", "every step", "the lifecycle" or "what happens when" of some operation (create/delete/upgrade/checkout/provisioning/sign-up...), wants a flow diagram or runbook-style write-up of how a request is processed, or points at a handler file and asks what it eventually does - even if they don't say "document" or "flow" explicitly.
---

# Code flow analysis

The goal is a write-up a teammate can trust: every step the system performs for one operation, every branch that changes the path, every externally visible effect (entities created, rows written, commands run on remote machines, API calls), in execution order. Two documents come out of it:

- **short** - steps only, readable in five minutes.
- **detailed** - the same steps under the same IDs, plus where the code lives, what is persisted, how failures behave, and what looks wrong.

The hard part is not writing, it is being *right*. A plausible flow doc that is subtly wrong is worse than none, because people will act on it. Most of this skill is about how to avoid being plausibly wrong.

## Phase 1 - Orient (do this yourself, don't delegate)

Read the entry point the user named, then follow the call chain until the work leaves the request: a row inserted for a poller, a message published, a workflow started, a goroutine spawned. Note what the synchronous part does (validation, records created, what is returned).

Then find the **engine** that drives the async part and read its shared machinery end to end, because every later step is an instance of it:

- how work is claimed (query, lock, lease, queue ack) and what makes an item eligible,
- how a step is dispatched (switch on stage/state, handler map, workflow registration),
- how success advances the state and how failure is recorded (is failure terminal? retried? who clears locks/flags?),
- retry policy (attempts, delays, per-type overrides, non-retryable errors),
- any feature flag that routes work to a different engine - and whether it actually applies to *this* flow.

Also read the project's CLAUDE.md / README / design docs. Treat them as hints, not truth: when docs and code disagree, the code wins, and the disagreement is itself worth a line in the detailed doc.

Output of this phase: a list of the **stages** (the big boxes) and a rough idea of which files own each one. Ask the user where to write the docs if they didn't say; a sensible default is `docs/<flow-name>/`.

## Phase 2 - Fan out, one subagent per stage

A real flow spans dozens of files; reading all of them yourself floods your context. Launch one subagent per stage (or per group of small stages) in a single message so they run in parallel. Use the brief in `references/subagent-brief.md` - it asks for exactly the facts both documents need, with `file:line` for every claim, and it asks for concrete artifacts rather than interface names ("writes `/etc/haproxy/haproxy.cfg` from `templates/haproxy.cfg.tmpl`", not "calls WriteFile").

While they run, keep working on the engine section of the detailed doc - you already have that material.

Fan-out pays off when the flow spans many stages and files. For a small flow (a request pipeline or a handful of files you can read in one sitting) read it yourself and skip this phase - the delegation overhead would cost more than it saves. Phase 3 still applies.

## Phase 3 - Verify before writing

Subagent reports are model output. They are usually good and occasionally confidently wrong, and two reports can disagree about the same boundary. Before writing:

1. **Build the transition chain from the code that writes the next state**, not from the order of enum declarations, switch cases or "requested stages" lists - those orders are often different from the real sequence. Grep for the assignments (e.g. `State: ...`, `NextState =`, `SetStage(`, `ExecuteActivity(` order) and check the union forms one unbroken chain from the initial state to the terminal one. Any state that nothing transitions into, or any branch with no exit, means something is missing or dead.
2. **Reconcile conflicts between reports.** A common one: one report quotes a constant's Go/Java name, another quotes its string value. Look it up.
3. **Spot-check the claims that matter most**, especially anything that will appear as a pitfall or "known issue", any flat claim about a specific line you haven't read, and any "this is dead code" claim. One grep or `sed -n` each. If a subagent hedged ("I did not check X"), either check X or keep the hedge - don't silently turn a hedge into a fact. Validation often lives in places reports don't look (generated contract validators, interceptors, DB constraints).
4. **Decide, for every remote command, whether it is exact or reconstructed.** If the command string is literally in this repo, it is exact. If this repo sends a structured RPC and another component builds the argv, say so - readers will grep for those strings on real machines.

If available, a second opinion (advisor/reviewer tool) before writing and again before declaring done catches the gaps you can't see.

## Phase 4 - Write both documents

Build **one canonical step list first** (IDs like `API-1..n`, then a short prefix per stage: `ENV-1`, `VM-1`, `M-1`...), then render it twice. Identical IDs and identical graphs are what make the pair useful - a reader skims the short one and jumps to the same ID in the detailed one.

Templates: `references/short-template.md` and `references/detailed-template.md`. Follow their structure; adapt section names to the domain.

ASCII graph rules (they keep graphs readable in a terminal and in any Markdown renderer):
- One top-level graph of the stages, then one graph per stage of its states. A single graph over 30+ states is unreadable and buries the branches.
- Boxes with `+---+`, arrows with `|`, `v`, `-->`; branch conditions as `<condition?>` with labelled `yes`/`no` legs; loops drawn back with `<---+`.
- Put the step ID on the arrow that performs it (`| ENV-3 create network`), so graph and text cross-reference.
- Plain ASCII only - no box-drawing Unicode, no emoji.

What every step needs (short version keeps the first four, detailed adds the rest):
- state in -> state out (and stage change, if any)
- what is created/changed, in which scope (project/account/namespace)
- every branch that changes the path, with the exact condition (field names)
- every command on a remote machine / external API call, exact or marked reconstructed
- status shown while the step runs, the handler/builder `file:line`, per-node/per-pool fan-out
- DB columns written, retry policy, criticality, what failure does
- string values of enums/task types as stored in the DB, when they differ from code names (people debug with SQL)

End the detailed doc with **Known issues and risks** - things found while reading (non-idempotent steps, lost guards across restarts, terminal failures, hard-coded limits, secrets at rest, misleading labels). Mark items that come from code reading only and were not traced end to end. This section is valuable precisely because nobody asked for it; keep it factual and sourced.

## Phase 5 - Report back

Tell the user where the files are, give the flow in a few bullets, and surface the handful of findings that would surprise them (a flag that doesn't do what they think, a failure that is terminal, a doc that's out of date). Say what is less certain (reconstructed commands, unverified items). Don't paste the documents into chat.

## Reference files

- `references/subagent-brief.md` - the per-stage research brief. Read before Phase 2.
- `references/short-template.md` - skeleton of the short document. Read before Phase 4.
- `references/detailed-template.md` - skeleton of the detailed document. Read before Phase 4.
