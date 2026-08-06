---
name: vibe-site-audit
description: >-
  Audits vibe-coded websites and web apps for common SEO, production, auth,
  Supabase, and security mistakes (empty view-source, default Vite/React titles,
  missing OG/sitemap/favicon, GA/GSC still showing React favicon, source maps,
  localStorage sessions, client-only admin checks, missing RLS, service-role
  leaks, no rate limits). Use when the user explicitly asks for vibe-site-audit,
  a vibe audit, vibe-coding checklist, site security/SEO pass, or production
  readiness review against a codebase or live URL.
disable-model-invocation: true
---

# Vibe site audit

Global personal skill (`~/.cursor/skills/vibe-site-audit`). Run a structured pass over a vibe-coded site or app. Check the **codebase** and, when a URL is given, the **live deploy**. Do not invent findings — cite file paths, routes, headers, or page evidence.

## Workflow

1. Identify target: repo root(s) and optional live URL(s).
2. Detect stack (Next, Vite/SPA, Supabase, Vercel, auth library).
3. Walk every checklist section below. Mark each item `pass`, `fail`, or `n/a` with evidence.
4. Deliver a **report only**: findings + an **optional fix plan** section inside that same report (prioritized written steps).
5. **Stop after the report.** Do not edit files, open PRs, or start implementing. The fix plan is text for the user to approve later — not a green light to code.
6. Severity: **critical** (exploit/data leak) → **high** → **medium** → **low** (polish/SEO).

Copy and track:

```
Vibe audit progress:
- [ ] SEO & discoverability
- [ ] Production polish
- [ ] Bundle & client hygiene
- [ ] Auth & sessions
- [ ] Backend / Supabase / data
- [ ] Secrets & deploy surface
- [ ] Scale readiness
```

---

## 1. SEO & discoverability

| Check | Fail if | Where to look |
|-------|---------|---------------|
| Meaningful HTML for crawlers | View-source / curl of HTML is empty shell (`<div id="root">` only) with no content | Live HTML, SSR/SSG vs CSR-only |
| Per-route titles | Same `<title>` on every page, or default `Vite` / `React App` / `Create Next App` | `index.html`, layout metadata, `Helmet`/`next/metadata` |
| Meta description | Missing or identical everywhere | `<meta name="description">`, metadata APIs |
| Open Graph image | No `og:image` (and ideally `twitter:card`) | Head / metadata |
| Structured data | No JSON-LD (or equivalent) where the page type warrants it (org, product, article, FAQ) | Head / page components |
| XML sitemap | No `/sitemap.xml` (or framework sitemap route); sitemap URLs blocked in `robots.txt` | `public/sitemap.xml`, Next sitemap, `robots.txt` |
| robots.txt | Blocks important paths, sitemap, or entire site unintentionally; missing sitemap directive | `public/robots.txt`, hosting headers |
| Custom 404 | Unknown routes show blank app shell or framework default with no real 404 | Router catch-all, `not-found`, hosting rewrites |
| Favicon present | No favicon linked in HTML / app metadata | `index.html`, layout metadata, `public/` |
| Favicon crawlable for GA/GSC | See **Favicon / Google still shows React icon** below | Live `/favicon.ico`, `public/favicon.ico`, `<link rel="icon">` |
| Custom domain | Live only on `*.vercel.app` (or similar) with no custom domain | DNS / Vercel project / canonical URLs |

### Favicon / Google still shows React icon

Common vibe-coding footgun: you set a custom favicon in HTML (or swapped a PNG in `public/`), but **Google Analytics / Search Console still show the default React (or Vite) icon**.

**Why:** Google often fetches **`/favicon.ico` at the site root**, not only your `<link rel="icon" href="...">`. CRA/Vite scaffolds leave a default `favicon.ico` in `public/`. If that file is still the scaffold icon while HTML points at `/logo.png` or `/icon.svg`, browsers may look fine and Google still shows React. Favicon caches at Google are also sticky.

**Audit steps:**
1. Open live `/favicon.ico` (and any linked icon URLs) — confirm bytes are *your* brand, not scaffold.
2. Confirm `public/favicon.ico` (or framework equivalent) was replaced, not only a secondary `<link>`.
3. Check `apple-touch-icon` / web manifest icons if present — same brand.
4. If code is correct but GA/GSC lag: note as **low** (cache); suggest re-fetch / wait after redeploy.

**Fail if:** no favicon, root `/favicon.ico` is still React/Vite default, or HTML icon and `/favicon.ico` disagree.

---

## 2. Production polish

| Check | Fail if |
|-------|---------|
| Console clean | Browser console full of errors/warnings on primary flows |
| No prod source maps | `.map` files shipped / `sourcemap: true` (or equivalent) in production build |
| Tab branding | Browser tab still says Vite / React / generic scaffold name |

---

## 3. Bundle & client hygiene

| Check | Fail if |
|-------|---------|
| Bundle size | Unreasonably huge JS for the product (no code-splitting, whole libraries imported, dead demo UI shipped) |
| Client errors | Uncaught exceptions, failed network calls left noisy in prod |

---

## 4. Auth & sessions (especially after “we added login”)

Treat a new login screen as a red flag that these five are missing unless proven otherwise:

1. **Session storage** — Session/JWT in `localStorage` / `sessionStorage` (XSS → full account takeover). Prefer httpOnly, Secure, SameSite cookies (or framework/auth provider defaults that do this).
2. **Authorization on server** — Admin/role checks only in React/client (hide button ≠ secure). Every privileged API/DB path must enforce authz server-side (RLS, middleware, edge functions).
3. **Identity proof** — No email verification and/or no 2FA where accounts matter. Anyone can sign up as anyone else’s email.
4. **Rate limiting** — Login, signup, password-reset, magic-link endpoints have no rate limit / lockout / CAPTCHA → brute force and email bombing.
5. **Password policy** — No minimum rules and no leaked-password check (HaveIBeenPwned / provider equivalent).

Also fail if:
- Password reset tokens never expire or are guessable
- Auth middleware only on some routes
- “Logged-in” UI state trusted without server session validation

---

## 5. Backend / Supabase / data

| Check | Fail if |
|-------|---------|
| RLS enabled | Supabase (or similar) tables exposed to `anon`/`authenticated` without RLS, unless intentionally public and documented |
| Service role sealed | `SERVICE_ROLE` / `service_role` key in frontend, `NEXT_PUBLIC_*`, client bundles, mobile apps, or public repos |
| Anon key scope | Anon key usable to read/write sensitive rows because RLS is off or policies are `true` |
| Public buckets | Storage buckets public that should be private; wide-open policies |
| IDOR | APIs accept arbitrary IDs without ownership checks |

---

## 6. Secrets & deploy surface

| Check | Fail if |
|-------|---------|
| Secrets in git | `.env` with real keys committed; secrets in client code |
| Public env misuse | Secret keys prefixed `NEXT_PUBLIC_` / `VITE_` / `PUBLIC_` |
| Verbose prod errors | Stack traces, SQL, or internal paths returned to clients |
| Security headers | No baseline headers where expected (CSP at least considered; no `X-Frame-Options`/`frame-ancestors` if clickjacking matters) |

---

## 7. Scale readiness (architecture, not premature micro-optimization)

| Check | Fail if |
|-------|---------|
| Indexes | Hot queries filter/join/sort on unindexed columns; lists scan whole tables |
| N+1 / unbounded reads | Per-row queries in loops; `select *` of huge tables; no pagination |
| “Works at 100 users” | No evidence of indexes, pagination, or query plans for growth past ~1k users |

Do not demand sharding. Do demand: indexes on foreign keys and common filters, pagination, and RLS-friendly query patterns.

---

## Extra common vibe-coding failures (check when relevant)

- CORS `*` with credentials or reflected origins
- Webhooks without signature verification
- File upload with no type/size/auth checks
- Paying features gated only in UI
- Debug/dev auth bypass left in prod
- Trusting `user_id` from request body instead of session
- Missing CSRF protections when using cookie sessions
- Dependency CVEs in lockfile left unattended on auth/crypto packages

---

## Output format

```markdown
# Vibe site audit: [site / repo]

## Summary
[2–4 sentences: overall risk and top blockers]

## Scorecard
| Area | Status | Notes |
|------|--------|-------|
| SEO & discoverability | pass/fail/partial | |
| Production polish | | |
| Bundle & client | | |
| Auth & sessions | | |
| Backend / Supabase | | |
| Secrets & deploy | | |
| Scale readiness | | |

## Findings
### Critical
- **[title]** — evidence → fix direction

### High
- ...

### Medium
- ...

### Low
- ...

## Optional fix plan
Written plan only (same report). Do not start fixing.
1. [critical first] — what to change, where
2. ...
```

Always include both **Findings** and **Optional fix plan** as report sections. Be specific: file paths, policy names, URL responses, header names. Prefer fewer high-confidence findings over a long speculative list. End the turn after the report unless the user explicitly asks to implement.
