# Vibecoded Site Audit Skill

Skill that audits vibe-coded websites. Same `SKILL.md` works in **Cursor**, **Claude Code**, and **Codex**. It scans your repo (and a live URL if you give one), checks the usual mistakes and vulns, and writes a report. It does **not** implement changes unless you ask.

## What it catches

Stuff that shows up again and again on AI-built sites:

- Empty view-source / CSR-only shells that look fine in the browser and thin to crawlers
- Default Vite / React tab titles, missing meta descriptions, no OG image
- Sitemap missing or "sitemap.xml" that is actually your SPA HTML because of a catch-all rewrite
- `/favicon.ico` still the React atom while your fancy PNG links look correct in Chrome (GA and Search Console love that one)
- Production source maps hanging out in public
- Login screens with tokens in `localStorage` / `sessionStorage`, client-only admin checks, no rate limits
- Admin/role checks only in React (hide button ≠ secure). Privileged API/DB paths need server-side authz (RLS, middleware, edge functions)
- Supabase with RLS off, or the service role key living somewhere the browser can see it
- Indexes and pagination that work at 100 users and fall over closer to 1,000

Plus a short list of extras when they apply (CORS wide open, UI-only paywalls, that sort of thing).

## Install

Clone this repo, then copy the `vibe-site-audit` folder into the skills directory for your tool. You want `…/vibe-site-audit/SKILL.md` on disk afterward.

### Cursor

**Personal (all projects)**

| OS | Path |
| --- | --- |
| Windows | `%USERPROFILE%\.cursor\skills\vibe-site-audit\` |
| macOS / Linux | `~/.cursor/skills/vibe-site-audit/` |

**Project-scoped:** `.cursor/skills/vibe-site-audit/` in the repo.

### Claude Code

**Personal (all projects)**

```text
~/.claude/skills/vibe-site-audit/SKILL.md
```

**Project-scoped:** `.claude/skills/vibe-site-audit/` in the repo.

Restart Claude Code after you drop the folder in (or start a new session) so it picks up the skill metadata.

### Codex

**Personal (all projects)** — either of these is common depending on your Codex version:

```text
~/.codex/skills/vibe-site-audit/SKILL.md
~/.agents/skills/vibe-site-audit/SKILL.md
```

**Project-scoped:** `.agents/skills/vibe-site-audit/` in the repo.

You can also install from GitHub inside Codex with `$skill-installer` and point it at this repo / the `vibe-site-audit` folder. Restart Codex if the new skill doesn’t show up right away.

Quick copy examples (macOS / Linux):

```bash
git clone https://github.com/sstanley-yelnatss/vibecoded-site-audit-skill.git
cd vibecoded-site-audit-skill

# Cursor
cp -R vibe-site-audit ~/.cursor/skills/

# Claude Code
cp -R vibe-site-audit ~/.claude/skills/

# Codex (pick the path your install uses)
cp -R vibe-site-audit ~/.codex/skills/
# or: cp -R vibe-site-audit ~/.agents/skills/
```

On Windows PowerShell, same idea with `Copy-Item -Recurse`.

## How to run it

Point it at a repo path and optionally a live URL.

**Cursor**

- `@vibe-site-audit` audit this repo
- use vibe-site-audit on https://yoursite.com

Cursor has `disable-model-invocation: true` on this skill, so naming it (`@vibe-site-audit` or “use vibe-site-audit”) is the reliable trigger.

**Claude Code**

- `/vibe-site-audit` against this repo
- or: run the vibe-site-audit skill on https://yoursite.com

The folder name becomes the slash command. You can also just describe the audit; Claude may load it from the description.

**Codex**

- `$vibe-site-audit` audit this repo
- or mention vibe-site-audit / ask for a vibe-coded site audit in plain language

## What you get back

A markdown report with:

1. A short summary
2. A scorecard by area
3. Findings by severity (critical → low), with evidence
4. An **optional fix plan** that is still just text in the report

The agent stops after the report. No silent refactors. If you want fixes, you say so in a follow-up.

## Repo layout

```text
vibe-site-audit/
  SKILL.md    # loaded by Cursor / Claude Code / Codex
README.md
LICENSE
```

## Notes

Open Graph tags are the preview cards Slack, LinkedIn, iMessage, and friends build when someone pastes your URL. Twitter/X cards are the same idea for X; most people set both. The skill checks for those.

Full crawlable HTML (SSR/SSG) is a bigger framework decision than this checklist. The skill will flag empty view-source; migrating off CRA/Vite SPA is on you if organic search is the goal.

## License

MIT. Use it, fork it, send it to the group chat.
