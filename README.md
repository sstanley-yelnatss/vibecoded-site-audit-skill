# Vibecoded Site Audit Skill

Skill that audits vibe-coded websites. It scans your repo (and a live URL if you give one), checks the usual mistakes/vulnerabilities, and writes a report. It does **not** implement changes unless you ask.

## What it catches

Stuff that shows up again and again on AI-built sites:

- Empty view-source / CSR-only shells that look fine in the browser and thin to crawlers
- Default Vite / React tab titles, missing meta descriptions, no OG image
- Sitemap missing or "sitemap.xml" that is actually your SPA HTML because of a catch-all rewrite
- `/favicon.ico` still the React atom while your fancy PNG links look correct in Chrome
- Production source maps hanging out in public
- Login screens with tokens in `localStorage`, client-only admin checks, no rate limits
- Supabase with RLS off, or the service role key living somewhere the browser can see it
- Indexes and pagination that work at 100 users and fall over closer to 1,000
- Session/JWT in localStorage / sessionStorage
- Authorization on server. Admin/role checks only in React/client (hide button ≠ secure). Every privileged API/DB path must enforce auth server-side (RLS, middleware, edge functions).

Plus a short list of extras when they apply (CORS wide open, UI-only paywalls, that sort of thing).

## Install

Personal install (works across all your projects):

1. Clone this repo somewhere convenient.
2. Copy the `vibe-site-audit` folder into Cursor's skills directory:

**Windows**

```text
%USERPROFILE%\.cursor\skills\vibe-site-audit\
```

**macOS / Linux**

```text
~/.cursor/skills/vibe-site-audit/
```

You want this on disk:

```text
~/.cursor/skills/vibe-site-audit/SKILL.md
```

Project install (shares with whoever clones the project): put the same folder at `.cursor/skills/vibe-site-audit/` inside the repo.

## How to run it

In Cursor/Claude/Codex, say something like:

- `@vibe-site-audit` audit this repo
- use vibe-site-audit on https://yoursite.com
- run a vibe site audit against `E:\path\to\site`

The skill is set to explicit invoke (`disable-model-invocation: true`), so naming it is the reliable way.

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
  SKILL.md    # the skill Cursor loads
README.md
```

## Notes

Open Graph tags are the preview cards Slack, LinkedIn, iMessage, and friends build when someone pastes your URL. Twitter/X cards are the same idea for X; most people set both. The skill checks for those.

Full crawlable HTML (SSR/SSG) is a bigger framework decision than this checklist. The skill will flag empty view-source; migrating off CRA/Vite SPA is on you if organic search is the goal.

## License

MIT. Use it, fork it, send it to the group chat.
