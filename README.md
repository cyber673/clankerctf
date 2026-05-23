# Agents & Skills for CTF — 101

A minimal skill + agent bundle for creating and solving CTF challenges
with AI coding agents, plus a bonus skill for theming CTFd.

Ships in two flavors so you can pick your runtime:

- **`claude/`** — for [Claude Code](https://docs.claude.com/en/docs/claude-code)
- **`codex/`** — for [OpenAI Codex CLI](https://developers.openai.com/codex/cli)

Companion repo for the **cyber673 × RE:HACK** session. Slides live in `slides/`.

> Demo target: `clankerctf.cyber673.com`

---

## Repo layout

```
agents-and-skills-for-ctf-101/
├── README.md                       <- you are here
├── slides/
│   └── agents-and-skills-for-ctf-101.pptx
├── claude/                         <- Claude Code format
│   ├── agents/
│   │   └── ctf.md
│   └── skills/
│       ├── ctf-fetch/SKILL.md      <- pull challenges from CTFd
│       ├── ctf-solve/SKILL.md      <- dispatch the ctf agent
│       ├── ctf-create/SKILL.md     <- author + push to CTFd
│       └── ctfd-design/SKILL.md    <- bonus: skin your CTFd
└── codex/                          <- Codex CLI format
    ├── README.md                   <- Codex-specific install + notes
    ├── agents/
    │   └── ctf.toml
    └── skills/
        ├── ctf-fetch/SKILL.md
        ├── ctf-solve/SKILL.md
        ├── ctf-create/SKILL.md
        └── ctfd-design/SKILL.md
```

The four `SKILL.md` files follow the [open agent skills standard](https://agentskills.io),
so they're functionally identical between the two folders. The agent
file differs (markdown for Claude, TOML for Codex).

---

## Install — Claude Code

```bash
git clone https://github.com/cyber673/clankerctf.git
cd clankerctf/agents-and-skills-for-ctf-101    # path may vary, adjust as needed

mkdir -p ~/.claude/skills ~/.claude/agents
cp -r claude/skills/*    ~/.claude/skills/
cp -r claude/agents/*.md ~/.claude/agents/
```

Open Claude Code anywhere — these should autocomplete:

```
/ctf-fetch
/ctf-solve
/ctf-create
/ctfd-design
```

## Install — Codex CLI

See `codex/README.md` for the Codex-specific paths (`~/.agents/skills`
for skills, `~/.codex/agents/` for the agent) and the small differences
between the two runtimes (slash vs `$` invocation, agent dispatch model).

---

## Quickstart — solve a CTF

You need a CTFd API token: log in to the CTFd instance →
**User Settings → Access Tokens → Generate**.

```
/ctf-fetch https://clankerctf.cyber673.com ctfd_xxx
```

Pulls every challenge into a workspace organized by category, with a
`README.md` + `solve.py` template per challenge and a master `INDEX.md`.

Then:

```
/ctf-solve
```

The `ctf` agent picks up the workspace and starts working through it.
It may clone itself to attack multiple challenges in parallel.

Scope it down if you don't want the full sweep:

```
/ctf-solve web
/ctf-solve web/login_bypass
```

---

## Quickstart — author a challenge

```
/ctf-create web easy command-injection
```

Produces a complete challenge folder: Dockerfile, source, `solve.py`,
writeup, and `challenge.yml`. Then pushes the challenge to CTFd via the
admin API (you'll need `CTFD_URL` and `CTFD_ADMIN_TOKEN` exported).

Local test before pushing:

```bash
cd <challenge_name>
docker compose up
python solution/solve.py    # should print the flag
```

---

## Bonus — design a CTFd theme

```
/ctfd-design cyber673 dark-terminal
```

Designs a CTFd v3.x theme. First produces a static HTML mockup so you
can iterate on the look without touching CTFd. Once approved, ports it
into a drop-in `themes/<your-theme>/` folder and uploads it via the
admin API.

Leans on the design principles in `~/.claude/skills/design-quality/` if
you have that skill installed.

---

## Session demo flow

The session runs **two separate Claude Code chats** to make the loop
obvious:

**Chat A — author**
1. `/ctf-create web easy command-injection`
2. `docker compose up` → confirm `solve.py` captures the flag
3. Push to `clankerctf.cyber673.com` via the skill's CTFd API workflow

**Chat B — solve** (clean context, doesn't know about Chat A)
1. `/ctf-fetch https://clankerctf.cyber673.com ctfd_xxx`
2. `/ctf-solve` (or scope to the new challenge)
3. Confirm flag submission in CTFd

Bonus, if there's time:
4. `/ctfd-design cyber673 dark-terminal` → restyle clankerctf

---

## Honest gotchas

- The `ctf` agent runs on `opus` by default. Drop to `sonnet` in the
  frontmatter if you want it cheaper/faster for easier challenges.
- These skills are deliberately 101-level. A full set used in the
  speaker's daily workflow includes category-specific specialists
  (`crypto-ctf`, `pwn-ctf`, `forensics-ctf`, `hardware-ctf`, `ics-ctf`)
  and a strategist agent (`strat`). Add those later if you want depth
  in one category.
- Don't trust the chat output for flag submission — submit to CTFd and
  let the platform tell you it's right.
- Auto-solving someone else's active CTF without permission is rude and
  often against the rules. Use these on your own events, sanctioned
  trainings, or platforms that explicitly allow tool-assisted play.

---

## License

MIT. Do whatever, just don't claim you wrote the cyber673 × RE:HACK part.
