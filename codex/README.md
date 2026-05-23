# codex/ — same skills, Codex CLI flavor

Mirror of the `claude/` bundle, adapted for OpenAI's
[Codex CLI](https://developers.openai.com/codex/cli).

Part of the **cyber673 × RE:HACK** session bundle.
Repo: [github.com/cyber673/clankerctf](https://github.com/cyber673/clankerctf)

## Why this works

Codex skills follow the [open agent skills standard](https://agentskills.io)
— same `SKILL.md` shape as Claude Code. The four skills here are 1:1
ports from the sibling `claude/skills/` folder, with `/` invocations
swapped for `$` (Codex's syntax) in the usage examples.

The agent file format **does** differ:

| | Claude Code | Codex CLI |
|---|---|---|
| Agent file | `~/.claude/agents/<name>.md` (markdown + YAML) | `~/.codex/agents/<name>.toml` (TOML) |
| Required fields | `name`, `description` | `name`, `description`, `developer_instructions` |
| Model field | `model: opus` | `model = "gpt-5.4"` |
| Tool allowlist | `tools: Read, Write, Bash...` | inherits from session (or set `sandbox_mode`) |
| Skill invocation | `/skill-name` | `$skill-name` |
| Spawn mechanism | `Agent` tool from parent | Parent prompts Codex to spawn by `name` |
| Parallel work | Agent self-clones via `Agent` tool | Codex orchestrator fans out; results consolidated |

## What's here

```
codex/
├── README.md                       <- you are here
├── agents/
│   └── ctf.toml
└── skills/
    ├── ctf-fetch/SKILL.md
    ├── ctf-solve/SKILL.md
    ├── ctf-create/SKILL.md
    └── ctfd-design/SKILL.md
```

## Install

```bash
# From the repo root (after git clone https://github.com/cyber673/clankerctf.git)
cd agents-and-skills-for-ctf-101    # path may vary

# User-level locations
mkdir -p ~/.agents/skills ~/.codex/agents

# Skills go here (Codex scans .agents/skills, NOT .codex/skills)
cp -r codex/skills/* ~/.agents/skills/

# Agent goes here (note: .codex, not .agents)
cp codex/agents/ctf.toml ~/.codex/agents/
```

Restart Codex (`exit` then `codex`) if anything doesn't show up.

To verify:

```
codex
> $ctf-fetch
> $ctf-solve
> $ctf-create
> $ctfd-design
```

If they autocomplete (or Codex acknowledges them on first use), you're set.

## Usage notes

**Skills.** Same workflow as the `claude/` bundle. Type `$ctf-fetch` to
invoke explicitly, or just describe what you want and let implicit
matching trigger (Codex matches on the `description` field).

**The ctf agent.** Spawn explicitly when needed:

```
> Spawn the ctf agent on this workspace and have it solve all
  unsolved challenges. Use 4 parallel workers max.
```

Codex's orchestrator will fan out, run them in parallel up to
`agents.max_threads` (default 6), and return a consolidated response.

**Config knobs (optional).** Drop in `~/.codex/config.toml`:

```toml
[agents]
max_threads = 4         # cap on concurrent ctf workers
max_depth = 1           # one level of nesting is plenty here
```

## Differences worth knowing

- **No tool allowlist on the agent** — Codex inherits the session's
  sandbox/tool surface. If you want to lock the agent down, set
  `sandbox_mode = "read-only"` in the TOML (won't work for ctf since
  it needs to write solve scripts).
- **Reasoning effort matters.** `model_reasoning_effort = "high"` is
  set for ctf because crypto/pwn need it. Drop to `"medium"` if cost
  is a concern and you only do web challenges.
- **No agent self-cloning.** In Claude the agent decides to spawn
  copies of itself. In Codex the *parent* (you, or the orchestrator)
  decides. Phrase the parent prompt to fan out explicitly when needed:
  "spawn one ctf per unsolved challenge".
- **Skills folder is `~/.agents/skills`, not `~/.codex/skills`.** Codex
  reuses the open standard's folder name. Agents stay under `~/.codex/`.

## When to prefer which runtime

| Workflow | Recommended |
|---|---|
| Need parallel fanout where the agent decides who to clone | Claude Code |
| Want raw speed on routine solves, integrating with VS Code | Codex CLI |
| Already have a Codex setup with MCP servers wired up | Codex CLI |
| Working in a team with a repo on GitHub | Codex (project skills travel with the repo via `.agents/skills`) |

Both runtimes can coexist on the same machine. The `SKILL.md` files
are identical; only the agent file format differs.

## References

- [Codex Skills docs](https://developers.openai.com/codex/skills)
- [Codex Subagents docs](https://developers.openai.com/codex/subagents)
- [Agent Skills standard](https://agentskills.io)
