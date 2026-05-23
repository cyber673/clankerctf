---
name: ctf-fetch
description: Bootstrap a local CTF workspace by downloading every challenge (description, files, hints) from a CTFd v3.x instance via its API. Use at the start of a CTF when the user shares a CTFd URL + API token, or says "fetch", "download", or "pull" challenges from a CTFd platform.
---

# /ctf-fetch — CTFd workspace downloader

Pull every challenge from a CTFd v3.x instance into a clean local workspace, ready for `/ctf-solve` to attack.

## Usage

```
/ctf-fetch <ctfd_url> <api_key>
/ctf-fetch https://clankerctf.cyber673.com ctfd_abc123...
```

If either arg is missing, prompt for it. The API key lives under **User Settings → Access Tokens** in CTFd.

## What to do when this fires

1. Validate inputs (URL reachable, token works against `/api/v1/users/me`).
2. Create a working directory named after the CTF host (`<host_slug>/`).
3. For each challenge returned by `GET /api/v1/challenges`:
   - `mkdir -p <category>/<challenge_slug>/`
   - Write `README.md` with description, hints, point value, connection info.
   - Download every file from `GET /api/v1/challenges/<id>` into the challenge dir.
   - Write a `solve.py` template (pwntools shell for pwn, requests shell for web, plain Python otherwise).
4. Write a master `INDEX.md` at the workspace root: total challenges, total points, per-category counts, a checklist.
5. Report the path + summary back to the user.

## Output layout

```
clankerctf_cyber673_com/
├── INDEX.md
├── web/
│   ├── easy_chal/
│   │   ├── README.md
│   │   ├── solve.py
│   │   └── handout.zip
│   └── ...
├── crypto/
├── pwn/
├── forensics/
└── misc/
```

## Done criteria

- Workspace exists, organized by category.
- Each challenge has `README.md` + `solve.py` template + handout files.
- `INDEX.md` summarizes the round.
- User gets the absolute path and counts in the reply.

## After fetching

Hand off to `/ctf-solve` to start solving.
