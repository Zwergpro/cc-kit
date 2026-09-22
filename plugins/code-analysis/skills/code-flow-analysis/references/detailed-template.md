# Detailed document template

Same step IDs and same graphs as the short document, plus: code locations, status while running, fan-out, DB columns, retries, failure behaviour, pitfalls, and an appendix mapping code names to stored values.

Per-step tables keep the facts scannable; put narrative (ordered commands, rendered config contents) below the table as numbered lists or code blocks.

````markdown
# <Operation> flow - detailed version

This is the same flow as `<flow>-short.md`, with the same step IDs. For each step it adds:
- the code that runs it (`file:line`)
- the status shown while it runs
- how work is fanned out (per node / per item / whole entity)
- the DB columns each step writes
- the retry policy
- pitfalls found in the code

Line numbers refer to the code at the time of writing.

## Contents
<numbered list of sections>

---

## 0. Big picture
<same graph as the short doc, with the owning directory next to each stage>

**Where the order comes from.** <which code defines the real sequence - usually the
next-state writes - and which tempting lists do NOT>.

**Engine / flags.** <which engine, which flags do or do not apply, doc drift found>.

## 1. Engine mechanics (common to every step)

### 1.1 Claim and prepare
<graph + the eligibility predicate quoted from code + what happens on preparation errors>

### 1.2 Unit-of-work records
<what is inserted per step, uniqueness/dedup rules, empty-step behaviour>

### 1.3 Execute
<graph: parallelism, success path in order, failure path, what is always cleaned up>

- **Retry policy.** <defaults, backoff type, per-type overrides, never-retried errors>
- **Is failure terminal?** <yes/no, and the only ways out>
- **Idempotency guards.** <where they live and whether they survive a restart>

---

## 2. API - synchronous part

```
<same chain graph as the short doc>
```

<interceptors/middleware, including generated contract validation - it often holds
 constraints the handler doesn't>

**API-1 <name>.** `<fn>` (`file:line`).
- <bullets>

... one bold paragraph per API step ...

> Pitfall: <e.g. not transactional; what a partial failure leaves behind>.

---

## 3. <STAGE 1>

Provider: `<file:lines>`.
<stage-wide facts: scope, credentials, criticality, parallelism, "no remote commands">

```
<same stage graph as the short doc>
```

### <P1>-1 <Title>

| | |
|---|---|
| State in, then out | `<A>` -> `<B>` |
| Child state | `<pool/node/item state>` |
| Status while running | `<STATUS>` |
| Builder | `<file:line>` |
| Task | <n>x `<TaskType>` per <unit>, handled by `<file:line>` |
| Retry | <default / override> |

- **External call:** <call, scope, credentials, read-back>.
- **On the machine** (exact | reconstructed):
  1. Write `<path>` from `<template>`. Rendered content: <key fields>.
  2. `<command>`
- **DB:** `<table.column>`, ...
- **Branch:** **if** `<condition>` ... **else** ...

<repeat per step; branches get "### <P1>-n Branch: <what>" with if/else sub-bullets>

---

## N. End state

| Object | Value |
|---|---|
| <entity> | <final fields> |
| <children> | <final states> |
| External | <everything created outside the DB> |

<why the engine won't pick it up again>

---

## N+1. Known issues and risks found during the analysis

These come from reading the code, not from runs.

1. **<Short title>.** <What, where (`file:line`), consequence.>
...

Items <k, m> come from code reading only and were not traced end to end.

---

## Appendix A. Work-unit types: code name and stored value

| Step | Code constant | Stored value |
|---|---|---|
| <P1>-1 | `<TaskTypeX>` | `<X_VALUE>` |
````
