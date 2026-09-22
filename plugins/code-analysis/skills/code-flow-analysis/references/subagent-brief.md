# Per-stage subagent brief

Fill in the placeholders and send one brief per stage, all in one message so they run in parallel. Use a general-purpose agent (it must be able to read whole files, not just excerpts).

Why the brief looks like this: the two documents need the same facts for every step, and reports that answer "calls WriteFile" instead of "writes `/etc/x.conf` from `templates/x.tmpl`" force a second research pass. Asking for `file:line` on every claim makes the report checkable in Phase 3.

---

```
Repo: <absolute repo path>. Read <CLAUDE.md / README / design doc> first for architecture.
Read-only analysis: do not modify files, do not run the full test suite.

Context (already established, don't re-derive):
- Entry point: <file> -> <service fn>. It creates <records> and leaves the item in <initial state>.
- Engine: <how work is claimed, dispatched, how success advances state, how failure is recorded>.
- This stage starts at <state/stage constant> and is owned by <directory/files>.

Task: produce an exhaustive, precise description of the stage <STAGE NAME>:
<list of the stage's files, builders/preparers, handlers/activities/workers it dispatches to,
 and the client libraries they call - as far as you know them; find the rest>.

Report, in order:
1. Every state in this stage, in execution order: which code runs (file:line), the status shown
   while it runs, the units of work created (type, one per what: node/pool/item/whole entity,
   parallel or sequential), the next state/stage/status written on success, and any child
   entity state (pool/node/line item) written.
2. EVERY branch that changes the path, with the exact condition (field names, flags, config
   keys). Also list the obvious branches that do NOT exist here (e.g. "HA flag is not read in
   this stage") - that is useful information.
3. Every externally visible effect, in order:
   - cloud/external API calls: what is created, in which scope/account/project, with which
     credentials, what is read back;
   - commands or file writes on remote machines: exact path on the machine, source template in
     the repo, the exact command string. Find out HOW it reaches the machine (SSH, agent RPC,
     cloud-init, k8s API, helm from the controller...) and say whether the command string is
     literally in this repo or built by another component from structured arguments;
   - Kubernetes/helm/other API objects created.
4. What is persisted: tables/columns, files, caches. Note secrets stored in plain text.
5. Retry policy per unit of work, idempotency guards (and whether they survive a restart),
   what happens on failure (terminal? retried? which flags/locks are cleared?).
6. The hand-off: exact constant(s) that move the flow to the next stage.
7. Anything that looks wrong: non-idempotent steps, dead code, swapped labels, inconsistent
   inputs, hard-coded limits. Separate "verified in code" from "inferred".

Cite file:line for every claim. Output dense structured Markdown notes, no filler.
If you could not verify something (code lives in another repo, generated code, etc.), say so
explicitly rather than guessing.
```
