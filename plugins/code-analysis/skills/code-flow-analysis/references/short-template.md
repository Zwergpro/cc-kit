# Short document template

Aim: someone reads it in five minutes and knows every step, every branch and every remote command. No `file:line`, no retry details - those live in the detailed doc under the same step IDs.

Replace `<...>`; drop sections that don't apply; rename stages to the domain.

````markdown
# <Operation> flow - short version

Steps only: what is created, every branch, every command on a remote machine.
The detailed version with the same step IDs is `<flow>-detailed.md`.

<One or two sentences on which engine runs this flow, and any flag that does NOT affect it
 even though readers would expect it to.>

## 0. Big picture

```
 <entry point>
        |
        v
 +-----------------+   sync, inside the request
 | API-1 .. API-n  |   <what the sync part does>
 +-----------------+
        |  returns <id>  (<initial state>)
        v
 ================= async: <engine> =================
        |
        v
 +--------------------------+
 | <STAGE 1>                |  <P1>-1 .. <P1>-n   <one-line summary>
 +--------------------------+
        |
        +-- <condition that skips stages>? --------------+
        v                                                 |
 ...                                                      v
 +--------------------------------------------------------------+
 | <terminal state>                                             |
 +--------------------------------------------------------------+
```

## 1. How one step runs

<ASCII graph of the engine loop: claim -> prepare -> execute -> success/failure -> next claim.
 State plainly whether failure is terminal.>

## 2. API (synchronous)

```
 API-1 <...> --> API-2 <...> --> ... --> API-n return
```

- **API-1** <decode: derived fields, defaults>
- **API-2** <validation: each check that returns an error>
- **API-3..** <records inserted, with initial states>
- <note if the sync part is not transactional>

## 3. Stage <STAGE 1>

<one line: e.g. "No remote commands; cloud API and DB only.">

```
 <STATE_A>
   | <P1>-1 <what it does>
   v
 <STATE_B>
   |
  <condition?>
   | yes                     | no
   v                         |
   <P1>-2a <...>             |
   v                         v
 <STATE_C> <-----------------+
   | <P1>-3 <...>
   v
 <STATE_D>, stage -> <NEXT STAGE>
```

- **<P1>-1** `<TaskType/Activity>`: <what is created, where, stored in which field>.
  - **if** <exact condition>: <extra work>.
- **<P1>-2** Branch on `<field>`:
  - **if true (<P1>-2a):** <...>
  - **else (<P1>-2b):** <...>
- **<P1>-3** On the machine:
  1. Write `<path>` from `<template>`.
  2. `<exact command>`
  3. **if** <failure condition>: `<cleanup command>`, then retry.

<repeat per stage>

## N. End state

- <final state/status/flags of the main entity and children>
- <why the engine will not pick it up again>
````
