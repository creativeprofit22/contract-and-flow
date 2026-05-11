# contract & flow

Two slash commands for coding agents that find the gaps your tests can't.

- **`/contract`** — checks that what your code *promises* matches what it actually *does*.
- **`/flow`** — clicks through your app end-to-end and finds journeys that break, disconnect, or go silent.

Both commands **audit only**. They don't edit your code. They create one prioritised task per gap so you (or another agent) can fix them one by one.

Built for GG Coder, but the prompts are plain markdown, so adapt them to Claude Code, Cursor, Aider, or any harness that supports custom commands.

---

## Before you run them

Please do not paste random slash commands off the internet into your agent and hit enter. Not these, not anyone's.

Open `commands/contract.md` and `commands/flow.md`, read what they actually do, and ideally get your own agent to explain them back to you in plain English and recreate them in your own commands folder. That way you know exactly what is going into your task pane and you have not handed a stranger a shell on your project.

These particular commands are audit only and will not edit your code. The next slash command you find online might not be so polite. Build the habit now.

---

## Why these exist

Most audits are either too vague to act on ("your code has technical debt") or too narrow to catch real problems (one-file linters). These two cover a specific blind spot:

> **The gap between what the codebase says it does and what it actually does.**

That gap is where silent bugs live. A type says `proxy?: string` but no function ever reads it. A button exists but its handler was never wired. The agent can schedule a post but the calendar never refreshes. None of that fails a test — it just quietly disappoints the user.

`/contract` finds it at the **code/type level**. `/flow` finds it at the **user-journey level**.

---

## `/contract` — promises vs. reality

### What it does

Scans interfaces, types, abstract classes, CLI flags, config schemas, public APIs, and documented features — then checks every promise against the implementation.

It looks for six kinds of gaps:

| Gap | Plain English |
|---|---|
| **IGNORED** | The type accepts this field, but no code ever reads it. |
| **PARTIAL** | Works in some paths, silently ignored in others. (The most dangerous kind — users assume it works everywhere.) |
| **STUB** | Method exists but is empty or throws "not implemented". |
| **DOCUMENTED-ONLY** | Listed in the README or `--help` but no code backs it. |
| **PHANTOM** | The code reads/returns a field the type doesn't declare. |
| **FACADE** | A whole class implements the shape — but every method is fake. |

For each gap, it decides one of three fixes:

- **WIRE** — the feature should work; connect it.
- **TRIM** — the contract was speculative; delete the field. (Often the right answer.)
- **DOCUMENT** — the undocumented behaviour is useful; add it to the type instead.

### When to run it

- After refactoring a type or interface.
- Before publishing a library or releasing a CLI.
- When you suspect a config option silently does nothing.
- When onboarding to a codebase and you want to know what's real vs. aspirational.

### Example usage

```
/contract
/contract VideoExportOptions
/contract src/cli
```

With no argument it infers scope from your recent git diff or active plan.

---

## `/flow` — user journeys, end-to-end

### What it does

Maps every user journey through your app (UI → IPC → backend → DB → events → back to UI) and finds where it breaks.

This one is grounded in code **and** in a real browser/app driver. Without a driver it would just be fuzzy grep, so the command starts with a feasibility gate:

- Detects whether your project is a Web SPA, Electron app, React Native app, or server/CLI.
- If it's server/CLI only → bails out cleanly and points you to `/contract` instead.
- If a driver isn't installed → **asks first** before installing anything (Playwright is ~200MB).
- Asks for your dev-server URL or app entry before tracing.

Then it clicks through and reports findings:

| Finding | Plain English |
|---|---|
| **BROKEN** | The mechanic literally doesn't work. |
| **DISCONNECTED** | The feature exists but no flow reaches it (orphan tab, dead route). |
| **ASYMMETRIC** | The agent can do it but the UI can't, or vice versa. |
| **DEAD-END** | The action succeeds but leaves the user stranded. |
| **SILENT** | Click → nothing visible happens for >200ms. |
| **STALE** | State changes but the related view doesn't refresh. |
| **DUPLICATE-PATH** | Same thing via two routes with different behaviour. |
| **ONE-WAY** | You can enter a state but can't exit/undo it. |
| **CONFUSING** | Labels or layouts unclear (opinion — needs your buy-in). |
| **EMPTY** | View has no zero-data state designed. |

### When to run it

- After shipping a new feature, before calling it done.
- When a user says "it kind of works but…"
- When the agent and the UI have diverged.
- Before a demo.

### Example usage

```
/flow
/flow scheduling
/flow draft creation
```

---

## How they work together

Run **`/contract`** first to clean up the type/API layer. Run **`/flow`** to verify the user-facing journeys on top of it. Both produce tasks in the same task pane, so you can work through Critical → High → Medium in order regardless of which command found the gap.

---

## Grounding the audit against real code

Both commands lean on **ken-mcp** — at audit time, not fix time.

When the auditor hits an unfamiliar library, framework hook, decorator, IPC pattern, or store convention, it looks up canonical usage in real public repos *before* deciding the implementation is broken. Two payoffs:

1. **Fewer false positives.** What looks like an IGNORED field or a STALE view may be wired through a standard pattern the trace missed. Verifying against real-world usage catches that before a task gets created.
2. **Tasks become recipes.** Once the auditor has seen the canonical pattern, it bakes that pattern into the task — actual call signature, import path, invalidation key, hook shape. The fix agent executes instead of re-investigating from a cold chat.

Lookups are anchored to repos active in 2026 so the agent can't fall back to stale patterns from its training data.

The fix agent only re-runs a ken-mcp lookup as a fallback, when the audit-time recipe is ambiguous. Most low-quality fixes come from agents pattern-matching on what they *think* an API looks like — grounding once, at the right moment, eliminates that entire class of mistake.

---

## Install — GG Coder

Drop the files into your global commands directory:

```bash
# Global (available in every project)
mkdir -p ~/.gg/commands
cp commands/contract.md commands/flow.md ~/.gg/commands/

# Or per-project
mkdir -p .gg/commands
cp commands/contract.md commands/flow.md .gg/commands/
```

Restart your agent. Type `/contract` or `/flow` and autocomplete should pick them up.

---

## Install — other harnesses

The command files are just markdown with a YAML frontmatter header. The body is the actual prompt. To port them:

### Claude Code

Drop into `~/.claude/commands/` or `.claude/commands/`. Claude Code uses the same `$ARGUMENTS` placeholder, so most of it works as-is. The `allowed-tools` line uses GG Coder's tool names (`tasks`, `Bash`, `Read`, `Write`, `Edit`, `Grep`, `Glob`) — adjust to Claude Code's tool names if needed. The `tasks` tool is GG Coder-specific; in Claude Code, replace task-pane calls with TodoWrite or just have the agent print the task list inline.

### Cursor

Paste the body of each `.md` into a Cursor rule or custom command. Strip the frontmatter — Cursor doesn't use it. Replace any GG-Coder-specific tool references (`tasks`) with "list the gaps in the chat" or your own task system.

### Aider / Continue / others

The prompts are pure instructions — copy the body, replace `$ARGUMENTS` with however your harness passes arguments, and drop any tool-name references that don't apply.

### Adapting the audit logic itself

The classification tables (gap types, severity tiers, finding types) are the actual value here. Even if you don't run these as slash commands, you can paste the body into a chat and say *"audit this codebase using these rules"* — it works.

---

## Conventions both commands follow

- **No edits.** Audit-only. Every fix is delegated to a separate task.
- **One task per gap.** Self-contained, with `file:line`, severity, and a concrete fix — so a fix agent in a fresh chat can execute it with no extra context.
- **Severity is honest.** PARTIAL gaps default to High. FACADE modules default to Critical.
- **Bail loud before guessing.** `/flow` refuses to run without a driver. `/contract` asks before proceeding blind. Neither will fake results.
- **Skipped gaps are reported.** If a gap is too ambiguous to write a concrete fix, it's marked Skipped with a reason — never silently dropped.

---

## License

MIT. Use them, fork them, rewrite them, sell a course about them. No warranty.

---

## Contributing

These are opinionated prompts that have been iterated against real codebases. PRs welcome — especially:

- New gap types or finding types that catch real bugs.
- Better classification tables.
- Ports to other harnesses (drop a folder under `harnesses/`).

Issues with concrete examples ("ran /contract on X, it missed Y") are more useful than abstract suggestions.
