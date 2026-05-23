---
name: ctf-create
description: Design and build a complete CTF challenge — files, Docker deployment, solution script, writeup, CTFd metadata — and push it live to a CTFd instance via the admin API. Covers web, crypto, forensics, reversing, and misc categories. Use when the user asks to create, design, build, author, write, or deploy a CTF challenge.
---

# $ctf-create — Author a CTF challenge and push it to CTFd

Build a complete, deployable CTF challenge and register it in CTFd via the
admin API. The flow is: generate locally → validate `solve.py` works → push
challenge metadata + files via the API → confirm live in the CTFd UI.

## Usage

```
$ctf-create <category> <difficulty> <vuln>
$ctf-create web easy command-injection
$ctf-create crypto medium rsa-low-exponent
$ctf-create forensics easy lsb-stego
$ctf-create misc easy base64-chain
```

If the user gives a fuzzier brief ("a beginner web challenge about JWTs"),
pick reasonable defaults and confirm before generating.

## Credentials

Need two env vars exported (or a `.env` in the CWD):

```bash
export CTFD_URL=https://clankerctf.cyber673.com
export CTFD_ADMIN_TOKEN=ctfd_xxx   # Admin Panel → Settings → Access Tokens
```

If either is missing, generate the challenge locally and tell the user
what they need to set before pushing.

## What you produce

Every challenge ships with:

1. **Challenge artifact** — the actual thing players interact with (web app, ciphertext, image, etc.).
2. **`Dockerfile` + `docker-compose.yml`** — if hosted. Pure handout otherwise.
3. **`solution/solve.py`** — programmatically captures the flag end to end.
4. **`solution/writeup.md`** — intended path, alternate solutions spotted, gotchas.
5. **`challenge.yml`** — CTFd metadata.
6. **`README.md`** — player-facing description, no spoilers.
7. **`flag.txt`** — the flag, format `flag{...}` or per the event's convention.

## Standard layout

```
<challenge_name>/
├── challenge.yml          # CTFd metadata
├── README.md              # Player-facing description
├── docker-compose.yml     # Deployment (if hosted)
├── Dockerfile
├── src/                   # Source files
├── dist/                  # Files handed to players
├── solution/
│   ├── solve.py
│   └── writeup.md
└── flag.txt
```

## Difficulty calibration

| Difficulty | Points  | Solve time     | Techniques                          |
|------------|---------|----------------|-------------------------------------|
| Easy       | 100-200 | < 30 min       | One well-documented vuln            |
| Medium     | 200-400 | 30 min - 2 hrs | Requires chaining or research       |
| Hard       | 400-600 | 2-6 hrs        | Multiple steps, custom exploits     |

## Workflow

1. Pick a vuln pattern appropriate for the category and difficulty.
2. Generate the artifact + a memorable but non-obvious `flag.txt`.
3. Write `solution/solve.py` first — if it can't capture the flag end-to-end, fix the challenge before moving on.
4. If hosted: write the Dockerfile and `docker-compose.yml`. Add `read_only`, drop capabilities, set resource limits.
5. Write the player-facing `README.md`, the `challenge.yml`, and `solution/writeup.md`.
6. Sanity check locally: `docker compose up` cleanly, `solve.py` works against the running container, no obvious unintended solutions.
7. **Push to CTFd via API** (see below).
8. Verify in CTFd UI: the challenge appears in the right category, files download cleanly, flag submission works.

## Push to CTFd (admin API)

CTFd v3.x uses Bearer tokens prefixed with `Token`. The full reference is at
`https://docs.ctfd.io/docs/api/redoc`.

### 1. Create the challenge

```bash
curl -sS -X POST "$CTFD_URL/api/v1/challenges" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d @- <<'JSON'
{
  "name": "<challenge_name>",
  "category": "<web|crypto|forensics|rev|misc>",
  "description": "<player-facing markdown from README.md>",
  "value": <points>,
  "type": "standard",
  "state": "hidden"
}
JSON
```

Capture the returned `id` — you need it for the next steps.

### 2. Attach the flag

```bash
curl -sS -X POST "$CTFD_URL/api/v1/flags" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"challenge": <id>, "content": "flag{...}", "type": "static"}'
```

### 3. Attach hints (optional)

```bash
curl -sS -X POST "$CTFD_URL/api/v1/hints" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"challenge_id": <id>, "content": "<hint>", "cost": 50}'
```

### 4. Upload handout files

```bash
curl -sS -X POST "$CTFD_URL/api/v1/files" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -F "challenge=<id>" \
  -F "type=challenge" \
  -F "file=@dist/handout.zip"
```

### 5. For hosted challenges, set connection info

CTFd stores it in the description, so update the challenge body to include
the host/port (or URL for web):

```bash
curl -sS -X PATCH "$CTFD_URL/api/v1/challenges/<id>" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"connection_info": "https://chal.example.com"}'
```

### 6. Flip to visible

```bash
curl -sS -X PATCH "$CTFD_URL/api/v1/challenges/<id>" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"state": "visible"}'
```

### 7. Verify

```bash
# Confirm the challenge is live
curl -sS "$CTFD_URL/api/v1/challenges" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" | jq '.data[] | select(.name == "<challenge_name>")'

# Submit the flag programmatically as a sanity check
curl -sS -X POST "$CTFD_URL/api/v1/challenges/attempt" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"challenge_id": <id>, "submission": "flag{...}"}'
```

A `200` with `data.status == "correct"` confirms end-to-end.

## Done criteria

- Challenge folder exists locally, complete with all artifacts.
- `docker compose up` brings the service up cleanly (if hosted).
- `solution/solve.py` captures the flag against the running service.
- Challenge exists in CTFd, in the right category, with handout files attached.
- Flag submission via API returns `correct`.
- One-line summary back to the user: name, id, points, URL.

## Notes

- This skill handles CTFd registration only. **Hosting the challenge service**
  (the container behind the URL) is out of scope here — use your own
  deployment skill/agent for that, or deploy to localhost for testing.
- For events with strict naming or categorization, pull the existing
  challenge list first (`GET /api/v1/challenges`) and match the convention
  before creating.
- If the CTFd instance has CSRF protection on the admin UI, the API token
  bypasses it — no extra headers needed.
