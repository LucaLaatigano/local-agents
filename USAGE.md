# Usage Guide: The Catalog in Each Tool

How to use the catalog's agents in Claude Code, opencode, Kiro CLI, Codex,
and Gemini / Antigravity CLI — globally (available everywhere) or per project.
General principles (reference, never copy) are documented in
[INTEGRATION.md](INTEGRATION.md); these are concrete recipes per tool, verified
against official documentation (July 2026).

> **Paths in this guide.** Examples use `~/Developer/local-agents` (or
> `/Users/you/Developer/local-agents` where the tool requires an absolute path).
> Replace it with wherever you cloned the catalog. Absolute paths to a user home
> directory do not exist on a teammate's machine or in CI — see the final section.

## Summary: Tool Mechanisms

| Tool | Mechanism | Global | Per Project | References Files? |
|---|---|---|---|---|
| Claude Code | Subagents `.md` with frontmatter | `~/.claude/agents/` | `.claude/agents/` | Via "Read X" instruction in body |
| opencode | Agents `.md` with frontmatter, or JSON | `~/.config/opencode/agents/` | `.opencode/agents/` | Yes: `{file:...}` in JSON prompt |
| Kiro CLI | JSON agents | `~/.kiro/agents/` | `.kiro/agents/` | Yes, native: `"prompt": "file://..."` |
| Codex | Hierarchical AGENTS.md + skills | `~/.codex/AGENTS.md` and `~/.codex/skills/` | `AGENTS.md` at repo root | AGENTS.md files concatenate automatically |
| Gemini / Antigravity CLI | Subagents `.md` + hierarchical AGENTS.md + skills | `~/.gemini/antigravity-cli/` | `.gemini/` and `AGENTS.md` at root | Yes: `.md` subagents and on-demand reading |

Cross-cutting rule: **the agent requires read permission** to load its definition
and skills on demand. Without `read`, you have a generic assistant with a nice name.

---

## Claude Code

### Global (available in all projects)

Create `~/.claude/agents/architect.md`:

```markdown
---
name: architect
description: Senior Architect mentor - guidance on architecture, tradeoffs, design reviews, and learning paths. Helpful first, challenging when it matters.
---

Read /Users/you/Developer/local-agents/architect/AGENTS.md and fully adopt it:
identity, judgment model, and escalation ladder. It inherits
/Users/you/Developer/local-agents/generalist/AGENTS.md — read that too.
Load the skills it references when their triggers fire.
```

- The file body is the subagent's system prompt; minimum frontmatter is
  `name` + `description` (the description decides when Claude delegates —
  write it focusing on the trigger, not marketing).
- Optional: `tools:` to restrict tools and `model:` to pin a model.
  The installer writes `tools: Read, Grep, Glob, WebFetch, WebSearch, Agent`
  for catalog agents with `access: read`; `full` access agents inherit everything.
- Invoked explicitly ("use the architect agent") or Claude delegates automatically
  when the task matches the description.

### Per Project

The same file in `.claude/agents/architect.md` inside the repo. Project-level
overrides global if they share a name.

### Bonus: Catalog skills are directly compatible

The catalog's `SKILL.md` files follow the Agent Skills specification that Claude
Code reads natively. To expose a standalone skill without going through an agent:

```bash
ln -s ~/Developer/local-agents/generalist/skills/verification ~/.claude/skills/verification
```

Watch out for name collisions with existing skills; if they collide, rename the
symlink and ensure the frontmatter `name:` still matches the folder name.

---

## opencode

### Global

Create `~/.config/opencode/agents/architect.md` (filename = agent name):

```markdown
---
description: Senior Architect mentor - helpful first, challenging when it matters
mode: primary
---

Read /Users/you/Developer/local-agents/architect/AGENTS.md and fully adopt it:
identity, judgment model, and escalation ladder. It inherits
/Users/you/Developer/local-agents/generalist/AGENTS.md — read that too.
Load the skills it references when their triggers fire.
```

- `mode: primary` = selectable as primary agent (Tab);
  `mode: subagent` = delegable only.
- Frontmatter also supports `model`, `temperature`, and `permission` (e.g.
  `permission: { edit: deny }` for query-only agents). The installer emits
  `permission: { edit: deny, bash: deny }` for catalog agents with `access: read`.

### JSON alternative with `{file:}` — direct prompt without read instruction

In `opencode.json` the prompt can BE the file:

```json
"agent": {
  "architect": {
    "mode": "primary",
    "description": "Senior Architect mentor",
    "prompt": "{file:/Users/you/Developer/local-agents/architect/AGENTS.md}"
  }
}
```

Tradeoff: `{file:}` injects the content of `AGENTS.md` as system prompt (does not
depend on the model "reading properly"), but does NOT resolve inheritance — the
generalist is not included. For agents that inherit (architect, senior-dev, ux-ui),
the markdown variant with a "read both" instruction is more faithful. Paths in
`{file:}` resolve relative to the configuration file.

### Per Project

Same markdown in `.opencode/agents/architect.md` in the repo. Additionally, opencode
reads `AGENTS.md` at project root as general rules — there you can point to the
catalog so EVERY project agent knows the conventions.

---

## Kiro CLI

### Global

Create `~/.kiro/agents/architect.json` (filename = agent name):

```json
{
  "name": "architect",
  "description": "Senior Architect mentor - helpful first, challenging when it matters",
  "prompt": "file:///Users/you/Developer/local-agents/architect/AGENTS.md",
  "resources": [
    "file:///Users/you/Developer/local-agents/generalist/AGENTS.md"
  ],
  "tools": ["read", "write", "shell"],
  "allowedTools": ["read"]
}
```

For consultative agents (`access: read` in the catalog: visionary and eng-manager),
the installer emits instead:

```json
  "tools": ["read"],
  "allowedTools": ["read"]
```

- `"prompt": "file://..."` is native Pattern A: absolute paths are used as-is,
  relative paths resolve from the config directory.
- `resources` preloads files as context — ideal for generalist inheritance.
  You can also preload skills with a glob
  (`"file:///Users/you/Developer/local-agents/architect/skills/**/*.md"`), but that
  breaks progressive disclosure: you pay the full context every time. Prefer
  preloading only the `AGENTS.md` files and letting skills load on demand with
  the `read` tool.

### Per Project

Same JSON in `.kiro/agents/architect.json` within the repo (scoped from that
directory down).

### Usage

```bash
kiro-cli agent list                    # verify it appears
kiro-cli chat --agent architect        # start with the agent
```

---

## Codex

Codex does not have "named agents": its mechanism is the hierarchy of `AGENTS.md`
(concatenated from the root down to your directory; the closest takes precedence)
plus skills. Two parts:

> If you want the full suite, `./install.py --all --tool codex` does this
> automatically: copies agent bodies to `~/.codex/agents/` and writes the roster into
> `~/.codex/AGENTS.md` between markers, preserving anything already there.
> Below is the manual recipe for wiring an individual agent by referencing the
> catalog instead of copying it.

### Global — Generalist as base for everything

In `~/.codex/AGENTS.md` (personal guidelines, applies everywhere):

```markdown
Adopt the reasoning model defined in
/Users/you/Developer/local-agents/generalist/AGENTS.md — operating loop, epistemic
rules, and reporting contract. Load the skills it references when their
triggers fire.
```

### Per Project — The role that repository needs

In `AGENTS.md` at repository root:

```markdown
For this repository, additionally adopt
/Users/you/Developer/local-agents/senior-dev/AGENTS.md (work as a peer, not an
assistant).
```

Because files concatenate (global + project), inheritance builds itself: generalist
as global baseline, specific role per repository.

### Skills — directly compatible, just like in Claude Code

Codex deprecated custom prompts in favor of Agent Skills — the SAME `SKILL.md`
spec as the catalog. Symlink and done:

```bash
mkdir -p ~/.codex/skills
ln -s ~/Developer/local-agents/generalist/skills/verification ~/.codex/skills/verification
ln -s ~/Developer/local-agents/architect/skills/tradeoffs ~/.codex/skills/tradeoffs
```

Codex loads only name+description of each skill and fetches content when it matches
the task (native progressive disclosure). Invoked explicitly with `/skills` or
`$name`, or implicitly by description. Codex also scans `~/.agents/skills/` — a
location designed to be shared across tools; if another CLI adopts it tomorrow, a
single set of symlinks serves all.

---

## Gemini / Antigravity CLI

Antigravity CLI (`agy`) uses Gemini models (such as Gemini 3.8 Flash and 3.1 Pro)
and provides native support for progressive discovery of Agent Skills and agents
defined in Markdown with frontmatter.

> If you want the full suite, `./install.py --all --tool gemini` does this
> automatically: copies bodies to `~/.gemini/antigravity-cli/agents/`, skills to
> `~/.gemini/antigravity-cli/skills/`, and writes the roster into
> `~/.gemini/antigravity-cli/AGENTS.md`.

### Global (available in all projects)

Create `~/.gemini/antigravity-cli/agents/architect.md`:

```markdown
---
name: architect
description: Senior Architect mentor - guidance on architecture, tradeoffs, design reviews, and learning paths. Helpful first, challenging when it matters.
model: inherit
---

Read /Users/you/Developer/local-agents/architect/AGENTS.md and fully adopt it:
identity, judgment model, and escalation ladder. It inherits
/Users/you/Developer/local-agents/generalist/AGENTS.md — read that too.
Load the skills it references when their triggers fire.
```

### Per Project

With `./install.py --all --tool gemini --project <repo>`, agents and skills are
copied to `.gemini/` and the roster is written to `AGENTS.md` at repo root.
Antigravity CLI automatically discovers the rules in `AGENTS.md` when working in
the directory.

### Skills

Catalog skills follow the `SKILL.md` specification (`name:` and `description:` in
frontmatter). Antigravity CLI discovers them automatically and only loads the full
content when the task requires it.

---

## Global or Per Project? Decision Criteria

| Situation | Choose |
|---|---|
| Your default way of working, always (generalist) | Global |
| Roles you use everywhere (architect, senior-dev, ux-ui) | Global |
| A shared team repo: conventions traveling with code | Per Project (and commit the config) |
| Caution: per-project config referencing `~/Developer/local-agents` | Works ONLY on your machine — for teams, vendor the catalog into the repo or use Pattern B |

That last row is the pitfall to remember: absolute paths to your user home do not
exist on a teammate's machine or in CI.

## Smoke Test: Verifying the agent actually loaded

After configuring, do not assume — verify (rung 4, not rung 2). Open the agent and
ask something it only knows if it read the catalog:

> "What is your ladder of evidence and which rung does 'done' require?"

If it responds with the 4 rungs (Observed > Tested > Read > Recalled) and "minimum
rung 3", it loaded. If it gives a generic answer about testing, the prompt did not
reach it — check `read` permissions and paths.
