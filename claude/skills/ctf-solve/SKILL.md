---
name: ctf-solve
description: Solve CTF challenges in the current workspace. Spawns the ctf agent to systematically work through pwn, web, crypto, forensics, reversing, or misc challenges. Use when the user wants to solve, attempt, or work on CTF challenges, points at a fetched CTF workspace, or shares a single challenge folder.
---

# /ctf-solve — Solve CTF challenges

Spawn the **ctf** agent to systematically solve challenges in the current workspace.

## Usage

```
/ctf-solve                              # solve all unsolved in current workspace
/ctf-solve web                          # all web challenges
/ctf-solve web/login_bypass             # one specific challenge
```

## What to do when this fires

1. Locate the workspace. Look for `INDEX.md` (from `/ctf-fetch`), a `challenges/` dir, or per-challenge `README.md` files. If none are obvious, ask the user for the path.
2. Identify scope: all challenges / one category / one challenge.
3. Spawn `ctf` via the `Agent` tool. Brief with:
   - **Workspace path**: absolute path to the CTF root.
   - **Scope**: which challenges are in scope, in priority order if multiple.
   - **Per-challenge artifacts**: paths to binaries, source, handouts, target host:port if remote.
   - **Deliverable**: flag(s) extracted, the working exploit script committed under each challenge dir, plus a brief writeup.
4. For multiple independent challenges, the agent may clone itself to work them in parallel — let it.

## Done criteria

- Flag captured for each in-scope challenge (saved to `flag.txt`), OR
- Honest "stuck because X" status with what was tried.
- Working exploit script (`solve.py` / `solve.sh`) saved per challenge.
- One-paragraph writeup of the technique used per solved challenge.

## Tip

Don't paste the flag into chat and trust it. Submit it back to CTFd to confirm before celebrating.
