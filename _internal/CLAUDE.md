# CLAUDE.md — ScanSquirrel .org (scansquirrel.org)

## What this is
The **product front door**. Where people get routed into the software: AI-style intake
conversation, client/Squirrel logins, "become a squirrel" recruiting.

**This is NOT the .com site.** See "Two sites, one brand" below — read that before touching
anything, because the two repos share CSS files and page names and are easy to confuse.

## Repository
- GitHub: https://github.com/maxferrigni/scansquirrel-org (public)
- Local clone: `~/Projects/scansquirrel-org/`
- CNAME: `scansquirrel.org` (GitHub Pages)
- Origin: forked from `scansquirrel-website/index9.html` — the .org direction was built as a
  standalone deploy to visualize .com vs .org side by side.

## Two sites, one brand — .com vs .org
| | `.com` | `.org` |
|---|---|---|
| Repo | `scansquirrel-website` | `scansquirrel-org` |
| Folder | `~/Projects/scansquirrel-website/` | `~/Projects/scansquirrel-org/` |
| Note | `_internal/CLAUDE.md` | `_internal/CLAUDE.md` (this file) |
| Job | **Advertise and find people** — marketing, pricing, territories, SEO | **Manage software access** — intake, logins, operator onboarding |
| Shape | Many pages, shared nav, territory pages | Two pages, no nav, conversational intake |

Max's framing (2026-08-04): *"two separate efforts (vectors) under the same logic. I might
merge them one day. Or I might keep them separate. I'm truly not sure."*

**Treat them as separate until told otherwise.** Do not "sync" a change from one to the
other unless asked. They deliberately diverge.

## Site Structure
- `index.html` — Full-screen splash front door. Logo, one intake box, rotating example
  prompts, then a scripted conversation (size → self/help → ZIP → guidance). Dark section
  below with the fires mission statement and "Become a squirrel." **Currently `noindex`.**
  All CSS is inline except `/css/base.css`.
- `scan/index.html` — Focused photo-scanning landing page. Same intake pattern, plus a
  header bar with Call/Email and Client/Squirrel logins. **Indexable**, has a canonical, and
  carries **Google Analytics + Google Ads conversion tracking**.
- `css/` — Copied from the .com repo. `base.css` holds all brand tokens and is the only
  stylesheet either page actually loads. `nav.css`, `footer.css`, `legal.css`, `scan.css`,
  `territory.css` are present but **currently unused here** — they came along with the copy.
- `css/README.md` — The shared CSS architecture doc (tokens, load order, migration rules).
  Written for the .com site; the token table and naming rules still govern both.

## Analytics / Ads
- `/scan/` has GA4 `G-M0DFY2Y7XE` and Google Ads `AW-17990603569`, including a
  `phone_conversion_number` swap for (213) 290-2327.
- `index.html` has **no** analytics at all. If you're comparing traffic between the two
  pages, that asymmetry will bite you.
- Note the .com repo uses a **different** tracking setup — GTM container `GTM-WCLRP7VB`.
  The two sites are not measured the same way.

## Known Issues (READ BEFORE DIAGNOSING)
### Intake data is NOT being captured (OPEN — as of 2026-08-04)
- `var LOG_URL = '';` is empty in **both** `index.html` and `scan/index.html`.
- Consequence: when a visitor completes the intake conversation, `logIt()` falls through to
  `console.log` and **the submission is lost**. Nothing is stored, emailed, or POSTed.
- This matters most on `/scan/`, which is the page wired for **paid ad traffic**. Ad clicks
  can complete the whole flow and leave no record.
- The payload shape is already settled and matches the app's planned `POST /api/leads`:
  `{ text, size, pref, zip, timestamp, source }`. `source` is `scansquirrel.org` on the
  homepage and `scansquirrel.org/scan` on the scan page.
- Fix is to set `LOG_URL` to a live endpoint (a Formspree URL is the stopgap the code
  comments anticipate; the real target is the web app's `/api/leads`).

### Guidance CTAs are dead links
- In `guidance()`, the final "Start scanning" and "Find local help" buttons are `href='#'`.
  The conversation completes and then goes nowhere.

## Related projects
- **ScanSquirrel web app** — `~/Projects/scansquirrel-web/` — the Clerk/`app.scansquirrel.com`
  target both login links point at, and the eventual home of `/api/leads`.
- **ScanSquirrel .com** — `~/Projects/scansquirrel-website/`
- **ScanSquirrel desktop app** — `~/Projects/ScanSquirrel/`

## Email / domain note
scansquirrel.com is a Google Workspace **user alias domain** of stockprecision.com. DKIM and
DMARC were fixed 2026-08-04. Mail from @scansquirrel.com now passes SPF/DKIM/DMARC. The
`hello@scansquirrel.com` links on both pages are the live inbound address.

## Session Log
*Newest first. Did / Next / Gotcha.*

### 2026-08-04
- **Did:** Discovered this repo exists during a website session — it was **not on the project
  map** and had **no folder note**. Added both. Read both pages and the CSS to write this
  note. Working tree clean, last commit Aug 1.
- **Next:** Decide what to build on .org. The open items are intake capture (`LOG_URL`) and
  the dead guidance CTAs.
- **Gotcha:** `/scan/` runs Google Ads conversion tracking but captures zero intake data —
  paid clicks can complete the funnel and vanish. Fix `LOG_URL` before spending more on ads.
