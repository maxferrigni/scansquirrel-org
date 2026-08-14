# CLAUDE.md — ScanSquirrel .org (scansquirrel.org)

## What this is
The **product front door**. One page, one text box. A visitor types a real question,
Claude answers it with live pricing, and the two exits are *sign up free* (customer) and
*Squirrel log in* (operator). Both exits land on **app.scansquirrel.org**.

**This is NOT the .com site.** See "Two sites, one brand" below and read it before touching
anything — the two repos share CSS files and page names and are easy to confuse.

## Start here for infrastructure
`~/Projects/scansquirrel-web/docs/ARCHITECTURE.md` is the single source of truth for
Railway, Clerk, Cloudflare, DEV vs PROD, and the deploy loop. **Don't restate it here.**
This note covers only what is specific to the .org site.

## Repository
- GitHub: https://github.com/maxferrigni/scansquirrel-org (public)
- Local clone: `~/Projects/scansquirrel-org/`
- Hosting: **GitHub Pages**, `CNAME` = `scansquirrel.org`. Push to `main` = deploy.
- No build step. Plain HTML, inline CSS/JS, one shared stylesheet.
- Origin: forked from `scansquirrel-website/index9.html`.

## Two sites, one brand — .com vs .org
| | `.com` | `.org` |
|---|---|---|
| Repo | `scansquirrel-website` | `scansquirrel-org` |
| Folder | `~/Projects/scansquirrel-website/` | `~/Projects/scansquirrel-org/` |
| Job | **Advertise** — marketing, pricing pages, territories, SEO | **Get people into the software** — Squirrel Talk, logins |
| Shape | Many pages, shared nav | One page, no nav |
| Traffic | Has it | Nearly none (homepage is `noindex`) |

Max's framing (2026-08-04): *"two separate efforts (vectors) under the same logic. I might
merge them one day. Or I might keep them separate."* **Treat them as separate.** Do not
sync a change from one to the other unless asked. They deliberately diverge.

`app.scansquirrel.com` was **deleted 2026-08-11**. Every login link on both sites points at
`app.scansquirrel.org`. If you find a `.com` app link anywhere, it is a bug.

---

## Squirrel Talk — the thing on the homepage

Named by Max ("like *girl talk*"). It is a real Claude conversation, not a script.

**Why it goes through the backend:** this site is static GitHub Pages and can never hold
`ANTHROPIC_API_KEY`. Every turn is proxied by the FastAPI app.

```
scansquirrel.org  --POST-->  <backend>/api/squirrel-talk/chat  --> Claude (claude-opus-5)
                                     |
                          squirrel_talk_turns (Postgres)  <- every turn logged
```

| Piece | Where |
|---|---|
| Client | `index.html`, the block starting at `var API_BASE` |
| Endpoint | `scansquirrel-web/backend/app/api/squirrel_talk.py` (public, **no Clerk auth**) |
| Log table | `squirrel_talk_turns`, created in `backend/app/db/seed.py` |
| Log viewer | app → **Settings → Squirrel Talk** (owner only) |

**Live pricing.** The prompt is fed the real rate card, read every 5 minutes from
`pricing_config`, `service_rates`, and `app_settings.batch_setup_fee` — the same tables that
price real jobs. Change a rate in the app and Squirrel Talk quotes the new number.

**Estimate card.** The model may emit a fenced ```estimate``` JSON block. The server strips
it from the prose and returns it typed; the browser renders it as a card. A malformed block
is dropped silently and the prose still goes out — a bad block must never cost the visitor
their answer. The block is stripped whether or not it parses, so raw JSON can't leak into
the chat. Card copy says *starting point, negotiated with your squirrel*.

**Four abuse ceilings, each stopping a different thing:**

| Ceiling | Value | Stops |
|---|---|---|
| Chars per message | 2,000 | Token bombs |
| Turns per conversation | 20 | One person looping forever |
| Requests per IP | 20 / 15 min | One person hammering |
| Turns per day (global, DB-backed) | 500 | IP rotation, and the bill |

The daily cap is the only one that survives a redeploy, and the only one that stops someone
rotating IPs. IPs are stored **hashed**, never raw.

**Off-topic:** the system prompt declines and redirects. Max reviewed a transcript of
porn/politics/trolling on 2026-08-09 and accepted the behaviour.

### Known gap — hardcoded DEV backend
`API_BASE = 'https://backend-development-4bb2.up.railway.app'` in `index.html`. The public
site talks to the **DEV** backend. Before this matters commercially: point it at the PROD
backend and add `https://scansquirrel.org` to PROD's `CORS_ORIGINS`. There is a comment at
the call site saying exactly this.

---

## The two exits

| Exit | Link | Audience |
|---|---|---|
| Hero button | `app.scansquirrel.org/sign-up?as=home` | Customer scanning their own photos |
| Top right | `app.scansquirrel.org/sign-in?as=squirrel` | Operator |
| Lost gallery link | `mailto:hello@scansquirrel.com` | Guest — **guests never get an account** |

`?as=` changes **wording only. It never sets a role.** The pages live at
`scansquirrel-web/frontend/src/app/sign-in|sign-up/[[...]]/page.tsx` and share
`AuthShell.tsx`, which reuses the .org brand tokens so the jump doesn't feel like leaving
the site. If the palette here changes, change it there too — the file says so.

**Roles** (full table in ARCHITECTURE.md): Owner / Squirrel / Home Squirrel / Guest. Home
Squirrel is not a separate role, it's a Squirrel who said "my own photos" at onboarding.
Guests have no login at all — they hold a `/g/<token>` link.

## Site structure
- `index.html` — the whole site. Splash, Squirrel Talk, dark mission section, squirrel
  recruiting at the bottom. **`noindex`.** All CSS inline except `/css/base.css`.
- `scan/index.html` — separate paid-ads landing page. Still runs the **old scripted
  intake** (size → self/help → ZIP → guidance), not Squirrel Talk. Indexable, canonical,
  GA4 + Google Ads conversion tracking.
- `css/base.css` — brand tokens; the only stylesheet either page loads. The other files
  (`nav.css`, `footer.css`, `legal.css`, `scan.css`, `territory.css`) came along in the copy
  from .com and are **unused here**.

## Analytics asymmetry (this will bite you)
- `/scan/` has GA4 `G-M0DFY2Y7XE` + Google Ads `AW-17990603569` with a phone-number swap.
- `index.html` has **no analytics at all**.
- .com uses a *different* setup entirely — GTM `GTM-WCLRP7VB`.

The three surfaces are not measured the same way. Don't compare their numbers.

## Open issues
| Issue | Where | Matters when |
|---|---|---|
| `API_BASE` points at DEV backend | `index.html` | Before .org sees real traffic |
| `LOG_URL = ''` — scripted intake captures nothing | `scan/index.html` | Now — this is the **paid ads** page; clicks complete the funnel and vanish |
| Legacy `logIt()` still in `index.html` | `index.html` | Dead weight; Squirrel Talk logs server-side instead |
| The free-100-scans offer is unenforced | promise on this site, nothing in the app counts | When someone scans 101 |
| Squirrel Talk says video needs a specialist | system prompt | Max has said Squirrels *do* scan video — unresolved |
| Clerk still on the development instance | Clerk | Signup emails read `[Development]` from `notifications@accounts.dev`. **Cannot be fixed from this repo.** A production instance exists but 403s — the other session owns it; see ARCHITECTURE.md → Clerk. Cutover forces every existing account to re-register. |

## Related
- **Web app / backend** — `~/Projects/scansquirrel-web/` — read `docs/ARCHITECTURE.md` first
- **.com marketing site** — `~/Projects/scansquirrel-website/`
- **Desktop app** — `~/Projects/ScanSquirrel/`

## Email / domain
`scansquirrel.com` is a Google Workspace **user alias domain** of `stockprecision.com`.
DKIM + DMARC fixed 2026-08-04; mail from @scansquirrel.com passes. SPF stays misaligned by
design — outbound leaves under the primary domain and DMARC passes on DKIM alignment alone.
`hello@scansquirrel.com` is the live inbound address used all over this site.

## Session Log
*Newest first. Did / Next / Gotcha.*

### 2026-08-14
- **Did:** Brought this note back in line with reality — it had drifted ten days and still
  described a scripted intake, `app.scansquirrel.com`, and no Squirrel Talk. Documented
  Squirrel Talk end to end, the two exits and `?as=` semantics, the abuse ceilings, and the
  analytics asymmetry. Pointed at `scansquirrel-web/docs/ARCHITECTURE.md` for infrastructure
  instead of duplicating it.
- **Next:** Repoint `API_BASE` at the PROD backend and add `scansquirrel.org` to PROD's
  `CORS_ORIGINS`. Decide whether `/scan/` gets Squirrel Talk or keeps its script.
- **Gotcha:** The `[Development]` stamp on signup emails is the Clerk **development
  instance** — no change in either website repo can fix it, and switching to a production
  instance means every existing account signs up again.

### 2026-08-10
- **Did:** Moved the rotating example prompts into the search box to save vertical space,
  then narrowed them to short questions only and removed the instruction placeholder
  entirely — the box now only ever shows a quoted question (`d9f5c31`).
- **Next:** —
- **Gotcha:** Never name a JS global `history`. `window.history` is `[LegacyUnforgeable]`,
  so `var history = []` silently does nothing and the first `.push()` throws. It cost a
  round of "I typed hello and nothing happened." The transcript array is `talkHistory`.

### 2026-08-09
- **Did:** Rebuilt the hero for customers first — free-scans invitation up top, squirrel
  recruiting at the bottom, `Squirrel log in` in the top right, and a one-line mailto for
  guests who lost their gallery link. Killed the client-login dead end.
- **Next:** End-to-end signup testing through scansquirrel.org rather than Railway URLs.
- **Gotcha:** Browser caching repeatedly showed stale pages after a Pages deploy. Cache-bust
  with `?cb=<anything>` before concluding a change didn't land.

### 2026-08-08
- **Did:** Built Squirrel Talk — backend proxy endpoint, DB logging, live rate card,
  estimate card, four abuse ceilings, owner-only log viewer in the app.
- **Next:** Watch the log for a week before tuning the prompt.
- **Gotcha:** Every message must be inserted *above* the reply box (`addToConvo()`), or the
  input strands itself mid-thread as the conversation grows.

### 2026-08-04
- **Did:** Discovered this repo during a website session — it was not on the project map and
  had no folder note. Added both.
- **Next:** Decide what to build on .org.
- **Gotcha:** `/scan/` runs Google Ads conversion tracking but captures zero intake data.
  Still true.
