# Quick Catalog Reference

Quick orientation: which agent to pick, how to install it, where everything is.
**The source of truth is the agent definitions** — each agent explains itself in its
`AGENTS.md`, and this reference guide does not duplicate it.

> Previously this guide summarized every agent and skill in detail: 729 lines,
> 38% of all documentation. It was a translated duplicate of the catalog, so it
> drifted — leaving descriptions of an obsolete `senior-dev` and omitting `dba`.
> Deep content now lives where it maintains itself. The history is in git.

---

## Which agent should I pick?

| Agent | Pick it when… | Skills |
|-------|---------------|-------:|
| [`generalist`](generalist/AGENTS.md) | There is no clear specialist. It is the base: verify before claiming, decompose, decide the next step. Every agent inherits this. | 3 |
| [`architect`](architect/AGENTS.md) | Decisions are expensive to reverse: boundaries, dependency direction, "should this split into services?". Also design reviews and teaching fundamentals. | 3 |
| [`senior-dev`](senior-dev/AGENTS.md) | Building or modifying application code alongside a peer. The stack comes from skills, not the agent. | 4 |
| [`dba`](dba/AGENTS.md) | Touching persisted data: schema, migrations, indexes, slow queries, integrity. It OWNS the model and sets limits for the rest. | 3 |
| [`ux-ui`](ux-ui/AGENTS.md) | Designing screens, flows, or states, and handing them off to development in implementable terms. | 4 |
| [`qa`](qa/AGENTS.md) | Finding what breaks: unconsidered paths, interruptions, edge cases. Writes tests, never fixes product code. | 3 |
| [`security`](security/AGENTS.md) | Auditing systems from an attacker's perspective to defend them: threat modeling, audits, remediation. | 3 |
| [`devops`](devops/AGENTS.md) | Moving from "works on my machine" to "runs reliably": pipelines, infra, observability, reversible deployments. | 3 |
| [`data-ml`](data-ml/AGENTS.md) | Data pipelines, ML models, or integrating an LLM into a product without being a one-off demo. | 3 |
| [`apple-dev`](apple-dev/AGENTS.md) | Swift / iOS / macOS — and mentoring you so you can solve it yourself. | 4 |
| [`product-manager`](product-manager/AGENTS.md) | Turning an idea into a prioritized backlog: what problem, for whom, in what order. | 3 |
| [`visionary`](visionary/AGENTS.md) | Needing brutally honest feedback on the product, or trimming an inflated roadmap. | 3 |
| [`gamification`](gamification/AGENTS.md) | A flow is dropping users and needs engagement — with an explicit ethical line. | 3 |
| [`stark`](stark/AGENTS.md) | Complex or stuck problems, zero-to-one development, or a crisis (outage, demo tomorrow). | 3 |
| [`eng-manager`](eng-manager/AGENTS.md) | Unsure which agent to pick, or the work crosses specialties. Routing is its job. | 3 |

> In doubt? Start with `eng-manager`: it knows all 15 agents and points you to the right one.

---

## Installing

```bash
./install.py --list                          # inspect the catalog
./install.py --status                        # check what is installed and if it is up to date
./install.py --all --tool claude --dry-run   # preview
./install.py --all --tool claude             # install the full suite
```

Tools: `claude`, `opencode`, `kiro`, `codex`, `gemini`, `shared`.

- **Install the whole suite together.** Agents reference each other; installing
  one standalone leaves handoffs pointing to missing agents.
- **For a team**: `--project ~/code/repo` vendors the configuration into the repo
  (`.claude/`, `.opencode/`, etc.) so it travels with the code. Commit it.
- **Codex** has no named-agent concept: the installer copies the bodies and
  writes a roster into its `AGENTS.md`, preserving anything already there.
- Each install stamps the catalog commit it copied from, so `--status` reports
  how many commits behind you are and what changed.

Full details in [INSTALL.md](INSTALL.md).

---

## How they connect

Three layers, all extracted directly from the files — nothing aspirational.
[GRAPH.md](GRAPH.md) has the diagrams:

- **Inheritance** — `generalist` provides the reasoning model; `senior-dev`
  adds the Peer Contract on top for peers. The installer inlines the parent's
  `CORE.md`, so each installed copy is self-contained.
- **Handoffs** — where work goes when it stops belonging to me. Written as
  ownership ("this belongs to X"), never as an invocation command: not all tools
  support invocation.
- **Skill references** — one skill references another instead of repeating it.
  This is the densest layer of the catalog.

---

## Where everything lives

| File | Purpose |
|------|---------|
| [AGENTS.md](AGENTS.md) | Catalog index and **catalog rules**. Read before adding an agent. |
| [README.md](README.md) | Catalog entry point: roster, structure, providers. |
| [GRAPH.md](GRAPH.md) | How everything connects. Data blocks are auto-generated. |
| [INSTALL.md](INSTALL.md) | `install.py` guide: flags, conflicts, projects, provenance. |
| [USAGE.md](USAGE.md) | Per-tool setup recipes, global and per-project. |
| [INTEGRATION.md](INTEGRATION.md) | Referencing the catalog instead of copying it, and the tradeoffs. |
| `install.py` | The installer. Zero external dependencies, Python 3.9+. |
| `validate.py` | Catalog validation checks + regenerates `GRAPH.md` blocks. |

---

## If you add or edit anything

1. **Read the rules** in [AGENTS.md](AGENTS.md) — there are 13 and each explains its
   rationale. Most frequently forgotten: agent identity never names technologies
   (that belongs in skills), and external skills are referenced by NAME, never by path.
2. **Run `./validate.py`** before committing. If you touched agents or skills,
   `./validate.py --write` regenerates `GRAPH.md`.
3. **This guide only changes if you add or remove an agent.** The rest lives in each
   agent's files, where it maintains itself. An automated check validates that the table
   above contains all 15 agents.
