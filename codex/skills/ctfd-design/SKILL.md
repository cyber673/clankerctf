---
name: ctfd-design
description: Design and apply a custom CTFd theme — colors, typography, layout, hero, leaderboard styling, challenge cards — and install it onto a live CTFd v3.x instance via the admin API. Produces a drop-in theme folder (Jinja2 templates + CSS + JS) plus pushes it remotely. Use when the user wants to skin, theme, restyle, or redesign their CTFd instance.
---

# $ctfd-design — CTFd theme designer + installer

Design a CTFd theme that doesn't look like every other CTFd, then install
it onto a live CTFd v3.x instance via the admin API. No SSH required.

This skill leans on `design-quality` for the aesthetic bar — same DON'Ts
apply: no generic AI cyan-on-black, no glassmorphism-by-default, no
rounded-card-grid-everything.

## Usage

```
$ctfd-design <brand-or-vibe>
$ctfd-design cyber673 dark-terminal
$ctfd-design retro-arcade neon-on-black
$ctfd-design corporate-clean enterprise-blue
```

## Credentials

```bash
export CTFD_URL=https://clankerctf.cyber673.com
export CTFD_ADMIN_TOKEN=ctfd_xxx
```

## What to do when this fires

1. **Brief the brand**. Pull whatever brand context exists (logo, colour palette, existing site, reference image). If nothing is given, ask for: primary colour, secondary colour, vibe in 3 words, typography preference.
2. **Pick a direction and commit**. Per `design-quality`, choose a clear aesthetic direction (brutalist, retro-futuristic, refined-minimal, editorial, etc.) and don't water it down.
3. **Generate the HTML mockup first**, before touching CTFd's template structure. One static `mockup/index.html` showing the challenge board with realistic challenge cards, the scoreboard, the user menu. No framework, no build step. Open it, iterate on the look until it's right.
4. **Port the approved mockup into CTFd's theme structure**. Copy `CTFd/themes/core-beta` as the starting point, then override.
5. **Package and upload to the live CTFd via API** (see below).
6. **Activate the theme** via the configs API.
7. **Verify** the live UI renders correctly.

## CTFd theme layout

```
themes/<your-theme>/
├── static/
│   ├── css/main.css
│   ├── js/main.js
│   └── img/logo.svg
├── templates/
│   ├── base.html              # layout shell (nav, footer)
│   ├── page.html
│   ├── challenges.html        # challenge board ← main canvas
│   ├── scoreboard.html
│   ├── users/
│   ├── teams/
│   └── components/
└── theme.json                 # name + version metadata
```

Most work happens in three places: `static/css/main.css`,
`templates/challenges.html`, and `templates/base.html`. Don't over-fork —
copy `core-beta`, override what matters, leave the rest.

## Design system starter

Define design tokens up top in `main.css` (OKLCH, per `design-quality`):

```css
:root {
  --brand:        oklch(0.78 0.22 145);
  --brand-soft:   oklch(0.78 0.22 145 / 0.15);
  --surface:      oklch(0.16 0.01 145);
  --surface-2:    oklch(0.22 0.01 145);
  --ink:          oklch(0.95 0.02 145);
  --ink-soft:     oklch(0.75 0.02 145);
  --accent:       oklch(0.80 0.18 30);

  --radius-card:  8px;
  --radius-pill:  9999px;

  --step--1: clamp(0.78rem, 0.7rem + 0.4vw, 0.9rem);
  --step-0:  clamp(0.95rem, 0.85rem + 0.6vw, 1.05rem);
  --step-1:  clamp(1.2rem,  1rem + 1vw,     1.5rem);
  --step-2:  clamp(1.8rem,  1.5rem + 1.5vw, 2.4rem);
  --step-3:  clamp(2.5rem,  2rem + 2.5vw,   3.8rem);

  --font-display: "Space Grotesk", "Inter", system-ui, sans-serif;
  --font-body:    "Inter", system-ui, sans-serif;
  --font-mono:    "JetBrains Mono", ui-monospace, monospace;
}
```

Tint your neutrals toward the brand hue, use a real type scale, pick fonts
that aren't Inter-by-default.

## Push to CTFd via API

CTFd ships theme upload over the admin API. The flow is:

### 1. Validate the theme locally

```bash
# Must have these files
test -f themes/<your-theme>/theme.json
test -f themes/<your-theme>/templates/base.html
test -f themes/<your-theme>/templates/challenges.html
test -f themes/<your-theme>/static/css/main.css

# theme.json shape (CTFd reads name + author)
cat themes/<your-theme>/theme.json
# { "name": "<your-theme>", "author": "cyber673", "version": "1.0.0" }
```

### 2. Zip and upload

```bash
cd themes
zip -r <your-theme>.zip <your-theme>/

curl -sS -X POST "$CTFD_URL/api/v1/themes" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -F "file=@<your-theme>.zip"
```

A `200` confirms the theme was unpacked into `CTFd/themes/<your-theme>/`
on the server.

### 3. Activate it

```bash
curl -sS -X PATCH "$CTFD_URL/api/v1/configs" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ctf_theme": "<your-theme>"}'
```

### 4. Verify

```bash
# Confirm the active theme switched
curl -sS "$CTFD_URL/api/v1/configs/ctf_theme" \
  -H "Authorization: Token $CTFD_ADMIN_TOKEN" | jq .

# Pull the rendered challenges page and grep for a unique marker from your CSS
curl -sS "$CTFD_URL/challenges" | grep -i "<your-unique-class>"
```

Hard refresh in the browser — CTFd doesn't cache-bust the theme switch on
its own.

### Fallback if the theme upload API isn't enabled

Some CTFd deployments lock the theme upload endpoint behind a config flag
(`THEME_FALLBACK = True` is required for full theme upload). If the upload
returns `403` or `400` with "theme upload disabled":

- Drop the theme into `CTFd/themes/<your-theme>/` on the host manually (SSH or volume mount).
- Then activate it via the configs API as in step 3.

## Done criteria

- `mockup/index.html` exists and opens with no build, looks like the brief.
- `themes/<your-theme>/` exists and is structurally complete.
- Local validation passes (all required files present).
- Theme uploaded to CTFd, activated, and the live challenges page renders with the new design.
- One-paragraph rationale doc explaining the colour choice, type choice, and the single thing that makes this theme distinctive.

## Cross-references

Lean on:

- `~/.claude/skills/design/SKILL.md` — for the HTML-mockup-first workflow.
- `~/.claude/skills/design-quality/SKILL.md` — for the aesthetic bar.

CTFd theme docs: `https://docs.ctfd.io/docs/themes/overview`.
CTFd API reference: `https://docs.ctfd.io/docs/api/redoc`.
