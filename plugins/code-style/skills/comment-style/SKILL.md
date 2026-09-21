---
name: comment-style
description: House style for comments and docstrings in any language — short, direct, one point each, explaining why rather than restating the code. Use this skill whenever writing or editing comments or docstrings in code, and whenever the user asks to clean up, tidy, shorten, rewrite, or review comments; mentions comment style, over-commented code, noisy or "AI-sounding" comments; or wants a comment pass over a file, diff, or PR. Applies to every language — the rules are about what a comment says, not about syntax.
---

# Comment style

One standard, two situations:

- **Writing** a comment in code you are producing. Start at [Style rules](#style-rules).
- **Cleaning up** comments that already exist. Same rules, plus the care needed not to break something — see [Cleanup pass](#cleanup-pass).

Nothing here depends on the language. A comment is prose sitting next to code, and prose fails the same way everywhere: it repeats what the code already said, it buries the point, or it is wrong.

## What a comment is for

The code already says *what* it does. A comment earns its place by adding what the code cannot: why this approach, what constraint forced it, what breaks if someone changes it, where the behavior surprises a reader.

That single idea decides most of the work.

```
i += 1              # increment i                          <- delete, restates the code
sleep(0.2)          # rate limit: server 429s above 5/s    <- keep, the code cannot say this
```

Before writing a comment, check whether a better name or a small extracted function removes the need for it. That is usually the stronger fix. Default to no comment; reach for one when the code genuinely cannot speak for itself.

## Style rules

**Short.** The fewest words that carry the full point. A comment is read far more often than it is written, usually by someone scanning.

**Lead with the action, finding, or decision.** Reason after, when it helps.

> `# We need to be careful here because the upstream service sometimes returns duplicates, so we deduplicate the results.`
> -> `# Dedupe: upstream repeats items on retry.`

**One point per comment.** Two unrelated things means two comments, each next to the code it describes.

**Concrete over abstract.** Name the actual constraint, value, or failure. "Handles the edge case" says nothing. "Empty batch returns 400" says everything.

**Plain workplace English.** No corporate slogans, no forced slang, no invented names for ordinary concepts, no jargon reached for to sound polished.

**Letters, digits and plain punctuation only.** No Unicode arrows, bullets, box-drawing, emoji, typographic quotes or special symbols. The only dash is "-" and the only arrow is "->".

**No filler.** Cut "It's worth noting," "Importantly," "Note that," "Basically," "Bottom line," "In order to."

**Direct and constructive.** No flattery, no blame, no scripted enthusiasm. Nobody needs `# Nice clean helper!` or `# This is a hack, sorry :(` — say what the hack works around instead.

**Nothing the situation didn't ask for.** No warnings, alternatives, or rhetorical questions invented to fill space. No trace of the conversation that produced the code: "as requested", "per the spec above", "you may want to" address a person who will not be reading the file.

**Describe the code, not the edit.** "Added null check", "Changed to use the new API", "Now handles empty input" narrate a diff; the commit message does that. A comment says what the code does today and why, as if it had always been that way.

**State uncertainty plainly.** "Unclear whether this holds when the cache is cold" is a fine comment. Presenting a guess as verified fact is not. If you are unsure what the code does, do not write a comment asserting that you are.

**Match the file.** If the surrounding comments are lowercase fragments without periods, yours is too. If they are in Russian, yours is too. House style beats personal style — consistency is what makes a file scannable.

### Docstrings and API docs

Same rules, plus:

- Keep the structural convention the file already uses. A comment pass is not the moment to convert between doc formats.
- Drop parameter lines that only restate the signature — a name and type repeated in prose earns nothing. Keep the ones carrying units, ranges, ownership, nullability, or failure modes.
- Keep the first line a one-sentence summary of what the thing is for.
- Never delete executable examples.

## Cleanup pass

### 1. Read before you cut

Read the comment *and* the code it describes. You cannot tell a redundant comment from a load-bearing one without knowing what the code does. This step prevents the expensive mistake: deleting the one comment that explained a workaround nobody can reconstruct.

### 2. Sort each comment into one of four buckets

| Bucket | What it looks like | Action |
|---|---|---|
| **Load-bearing** | Explains why, warns about a constraint, cites a bug or spec, documents a non-obvious contract | Keep. Tighten the wording only if it stays just as clear. |
| **Redundant** | Restates the next line, narrates obvious structure ("loop over users", "Step 1: parse input"), banners around self-evident code, a docstring that repeats the function name | Delete. |
| **Wrong** | Contradicts the code — describes a parameter that no longer exists, a return value that changed | Fix it when the correct content is obvious from the code. When it isn't, leave it and say so in your summary. A confidently wrong comment is worse than none. |
| **Verbose** | Right content, three sentences of throat-clearing around it | Rewrite per the style rules. |

### 3. Never touch these

Some things are written as comments but function as code. Editing them changes behavior, and the damage is silent — the build breaks later, or worse, it doesn't.

- **Tool directives**: linter and type-checker suppressions, coverage and formatter toggles, build tags, bundler and compiler hints. Anything of the shape `noqa`, `type: ignore`, `fmt: off`, `disable-next-line`, `nolint`, `ignore`, `pragma`, `expect-error`. If a comment looks like it is addressed to a machine rather than a person, leave it exactly as it is.
- **Interpreter and encoding lines** at the top of a file.
- **License and copyright headers.**
- **Generated-file markers** (`DO NOT EDIT`, `@generated`).
- **`TODO`/`FIXME` carrying a ticket reference or owner.** You may tighten the prose after the reference; never drop the reference.
- **Links** to issues, specs, RFCs, CVEs, or the discussion that explains the code.
- **Executable examples** inside docs, and any region or fold marker tooling depends on.
- **Commented-out code.** It is not prose and deleting it is a code-review decision, not a style one. Leave it and list it in your report.

Docstrings deserve one extra step of care. In Python they are runtime data: `help()`, doctest, and web frameworks that build API descriptions from them all read `__doc__`. Rewrite a bad docstring rather than deleting it, unless you have checked that nothing consumes it.

When in doubt whether a comment is load-bearing, keep it. The cost of keeping a mediocre comment is a line of noise. The cost of deleting a load-bearing one is a bug nobody can explain six months later.

### 4. Never modify a non-comment line

This is the hard invariant. A comment pass that reformats code, renames a variable, or fixes a bug it noticed along the way is no longer a comment pass — it is a diff nobody can review. Spotted a real bug? Finish the comment work and mention it in your summary.

One exception: when deleting a comment leaves a stray blank line that reads badly, removing that blank line is fine. Nothing else.

### 5. Verify

Run the bundled checker on every file you changed:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/check_comment_edit.py" <file>
```

Claude Code fills in that path when it loads this skill. If it shows up literally, the script is in `scripts/` next to this SKILL.md — resolve it from there, not from the project root.

With one argument the checker takes the original from `git HEAD`. That is only right when the file was clean before you started, so run `git status --short` on the files *before* editing. For a file that was already modified or untracked, copy it aside first and pass both: `check_comment_edit.py <copy> <file>`.

It looks only at the changed lines. Each must be blank, a whole-line comment, or a code line whose code part is unchanged, so any code touched by accident shows up as a failure. It also confirms tool directives, license headers, and references in removed comments survived, and reports how much comment text went. Fix anything it flags before reporting done. A note about removed docstrings is not a failure, but it is a question: confirm nothing reads them.

The checker knows the comment markers of common languages and falls back to `#`, `//` and `/* */` otherwise. It is a safety net, not a parser: if it flags a line you are sure is a comment, look at the line, then decide.

### 6. Report

A few lines: how many comments deleted, rewritten, kept. Call out what you deliberately left alone and why — a wrong comment you couldn't safely fix, an awkward workaround note you kept because the content mattered. Don't paste the whole diff back.

## Examples

Comment markers below are illustrative; the point is the prose.

**Redundant — delete**

```
// NewClient creates a new client.
func NewClient() *Client
```
The name already says it. Delete it — unless the project requires a doc comment on every exported symbol, in which case keep the convention and make it earn its place: `// NewClient returns a client with a 30s default timeout.`

**Verbose — rewrite**

```
// It's worth noting that we are using an ordered map here instead of a plain
// dictionary because we need to preserve the insertion order of the keys, which
// is important for the way the downstream renderer consumes this data.
```
-> `// Ordered map: the renderer depends on insertion order.`

**Wrong — fix**

```
# Returns a list of user IDs.
def get_users(...) -> list[User]
```
-> Delete it. Corrected, it would only restate the signature. If the project requires a docstring here, write one that adds something: `"""Users visible to the caller, active first."""`

**Load-bearing and ugly — keep the content, fix the words**

```
// NOTE: DO NOT refactor this into a comprehension!!! It looks like you can
// but the generator gets consumed twice and you get silent empty results.
```
-> `// Not a comprehension: the generator is consumed twice, silently yielding nothing.`

**Nothing to add — write nothing**

```
total = price * quantity
```
No comment. Any sentence here would restate the line.
