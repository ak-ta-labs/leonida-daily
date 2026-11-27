# Leonida Buzz / The Leonida Daily — v1 Project Tracker

Repo: `ak-ta-labs/leonida-daily` · Design reference: approved screenshot + `designHTML.html` / `styles.css`

**Status key:** ⬜ To do · 🟨 In progress · 🟦 In review · ✅ Done · ⛔ Blocked

---

## Ownership at a glance

| Area | Owner | Reviewer |
|---|---|---|
| `n8n/` — ingestion, dedup, LLM scoring & satire generation, publishing the feed | **Tornike** | Alexandra |
| Data contract (`feed.json` shape), scoring logic, pipeline ops & alerts | **Tornike** | Alexandra |
| `web/` — site build from the approved design, filters, search, states, responsive, a11y | **Alexandra** | Tornike |
| Satire editorial rules, source list, content QA, hosting & deploy | **Alexandra** | Tornike |
| `docs/` | Whoever owns the topic | The other person |

**Working agreement**
- One branch per task: `feat/<task-id>-short-name` (e.g. `feat/W-03-rank-card`).
- Every PR is reviewed by the other person before merge; no direct pushes to `main`.
- Reference the task ID in the PR title. Update the status column here in the same PR.
- `feed.sample.json` is the shared handshake — web builds against it, the pipeline must produce it. Changes to the contract need both approvals.
- No API keys in the repo. Credentials live in n8n only.

---

## Phase 0 — Setup

| ID | Task | Owner | Reviewer | Depends on | Status |
|---|---|---|---|---|---|
| S-01 | Scaffold repo: `web/`, `n8n/`, `docs/`, root `README.md`, `.gitignore`, `.editorconfig` | Alexandra | Tornike | — | 🟦 |
| S-02 | Branch protection on `main`, PR template, issue labels (`web`, `n8n`, `docs`, `bug`, `v2`) | Tornike | Alexandra | S-01 | ⬜ |
| S-03 | Pick web hosting (GitHub Pages / Netlify / Vercel) with preview deploys per PR | Alexandra | Tornike | S-01 | ⬜ |
| S-04 | Pick n8n hosting (n8n Cloud vs self-hosted Docker) and storage for seen-items + history (Google Sheet vs Supabase) — write decision in `docs/decisions.md` | Tornike | Alexandra | — | ⬜ |
| S-05 | Set up credentials in n8n: LLM API key, YouTube Data API key, GitHub/storage token | Tornike | — | S-04 | ⬜ |

## Phase 1 — Contracts & rules (everything downstream builds on these)

| ID | Task | Owner | Reviewer | Depends on | Status |
|---|---|---|---|---|---|
| C-01 | **Data contract** `docs/data-contract.md` + `web/data/feed.sample.json`. Buzz item: `id, rank, title, creator, category, tags[], platform, url, thumbnail, duration, views, trend_pct, verified, score, published_at`. Dispatch item: `id, headline, blurb, label, created_at`. Feed root: `generated_at, buzz[], dispatch[]` | Tornike | Alexandra | — | ⬜ |
| C-02 | **Category taxonomy**: Clips · Meta/Tips · Patch/News · Community → tag names + colors (Community has no color in current CSS — add one) | Alexandra | Tornike | — | ⬜ |
| C-03 | **Scoring spec** `docs/scoring.md`: hype score inputs (recency decay, views/engagement, source weight, LLM relevance), what "VERIFIED BUZZ" means, how `trend_pct` is computed between runs | Tornike | Alexandra | C-01 | ⬜ |
| C-04 | **Satire guidelines** `docs/satire-guidelines.md`: tone, always fictional, no real creators/people as targets, no real tragedies, mandatory "SATIRICAL PARODY" label, disclaimer copy | Alexandra | Tornike | — | ⬜ |
| C-05 | **Source list** `docs/sources.md`: verified RSS URLs (PC Gamer, GameSpot, Polygon, Eurogamer), Rockstar Newswire, YouTube search terms/channels, community source (e.g. r/GTA6), GTA VI keyword filter list | Alexandra | Tornike | — | ⬜ |

## Phase 2a — Pipeline (`n8n/`)

| ID | Task | Owner | Reviewer | Depends on | Status |
|---|---|---|---|---|---|
| N-01 | Schedule Trigger (every 6h) + Manual Trigger for testing | Tornike | Alexandra | S-04 | ⬜ |
| N-02 | RSS ingestion for all feeds in C-05, filter to GTA VI keywords | Tornike | Alexandra | C-05, N-01 | ⬜ |
| N-03 | YouTube ingestion: search + video details (views, duration, thumbnail, channel) | Tornike | Alexandra | C-05, S-05 | ⬜ |
| N-04 | Community source ingestion (Reddit or equivalent), mapped to `platform: "Reddit"` | Tornike | Alexandra | C-05 | ⬜ |
| N-05 | Normalize all sources to one item shape | Tornike | Alexandra | C-01, N-02–N-04 | ⬜ |
| N-06 | Dedup against storage by canonical URL / source ID; store first-seen time + view snapshot | Tornike | Alexandra | S-04, N-05 | ⬜ |
| N-07 | LLM node with structured JSON output: relevance, category, short summary, score inputs. Validate against schema, retry/skip on bad output | Tornike | Alexandra | C-02, C-03, N-06 | ⬜ |
| N-08 | Satire prompt `n8n/prompts/satire.md` (3–5 dispatch items per run, follows C-04) | Alexandra | Tornike | C-04 | ⬜ |
| N-09 | Wire satire LLM step into the workflow with structured output + label enforcement | Tornike | Alexandra | N-07, N-08 | ⬜ |
| N-10 | Ranking step: final hype score, `trend_pct` vs previous snapshot, top N | Tornike | Alexandra | C-03, N-07 | ⬜ |
| N-11 | Publish `feed.json` (commit to repo via GitHub node or push to storage bucket) — must match C-01 | Tornike | Alexandra | N-09, N-10 | ⬜ |
| N-12 | Golden test set `n8n/tests/golden-items.json`: ~20 real items with expected category to check N-07 accuracy | Alexandra | Tornike | C-02 | ⬜ |
| N-13 | Error workflow + failure notification; token/cost cap per run | Tornike | Alexandra | N-11 | ⬜ |
| N-14 | Export workflow JSON to `n8n/` (credentials stripped) + `n8n/README.md` import guide | Tornike | Alexandra | N-11 | ⬜ |

## Phase 2b — Website (`web/`)

| ID | Task | Owner | Reviewer | Depends on | Status |
|---|---|---|---|---|---|
| W-01 | Turn Gemini HTML/CSS into `web/` (`index.html`, `styles.css`, `app.js`); remove hard-coded cards, render from `feed.sample.json` | Alexandra | Tornike | C-01 | ⬜ |
| W-02 | Top bar: logo, nav (Rankings, The Dispatch anchor; Creators/Analysis hidden or "soon" for v1), search, right-side icons (missing in current CSS) | Alexandra | Tornike | W-01 | ⬜ |
| W-03 | Rank card: badge, thumbnail + duration, creator, title, tags, platform label, views, verified badge, trend %, source link (new tab, `rel="noopener"`) | Alexandra | Tornike | W-01, C-02 | ⬜ |
| W-04 | Filter pills as real `<button>`s with `aria-pressed`, filtering by category, state kept in URL hash | Alexandra | Tornike | W-03 | ⬜ |
| W-05 | Search: client-side filter by title + creator | Alexandra | Tornike | W-03 | ⬜ |
| W-06 | Dispatch panel: masthead, SATIRE badge, disclaimer, per-item "SATIRICAL PARODY" label (in screenshot, not in Gemini CSS) | Alexandra | Tornike | W-01, C-04 | ⬜ |
| W-07 | States: loading skeleton, empty state per filter, feed-failed error, broken-thumbnail fallback | Alexandra | Tornike | W-03, W-06 | ⬜ |
| W-08 | Responsive pass (1060px / 720px breakpoints + phone check) | Alexandra | Tornike | W-03, W-06 | ⬜ |
| W-09 | Accessibility: focus styles (search has `outline: none`), contrast of muted text, alt text, landmarks | Alexandra | Tornike | W-08 | ⬜ |
| W-10 | "Last updated" timestamp from `generated_at` | Alexandra | Tornike | W-01 | ⬜ |

## Phase 3 — Integration & QA

| ID | Task | Owner | Reviewer | Depends on | Status |
|---|---|---|---|---|---|
| I-01 | Point site at live `feed.json` URL, cache-busting, handle stale feed | Alexandra | Tornike | N-11, W-07 | ⬜ |
| I-02 | End-to-end run: trigger → feed → live site; validate output against contract | Tornike | Alexandra | I-01 | ⬜ |
| I-03 | Content QA over 3 real runs: categorization vs golden set, satire labeling, broken links/thumbnails, duplicates | Alexandra | Tornike | I-02, N-12 | ⬜ |
| I-04 | Tune scoring weights and prompts from QA findings | Tornike | Alexandra | I-03 | ⬜ |

## Phase 4 — Launch

| ID | Task | Owner | Reviewer | Depends on | Status |
|---|---|---|---|---|---|
| L-01 | Footer: satire disclaimer, "not affiliated with Rockstar Games / Take-Two", source attribution | Alexandra | Tornike | C-04 | ⬜ |
| L-02 | Check thumbnail/content usage against YouTube API and RSS source terms (link back, no re-hosting) | Tornike | Alexandra | N-03 | ⬜ |
| L-03 | Production deploy + domain | Alexandra | Tornike | I-03 | ⬜ |
| L-04 | Monitoring: alert if a run fails or feed is older than 12h | Tornike | Alexandra | N-13 | ⬜ |
| L-05 | Root `README.md`: overview, architecture diagram, run web locally (Alexandra's section), import & run n8n (Tornike's section) | Both | — | N-14, L-03 | ⬜ |
| L-06 | Launch checklist walkthrough together | Both | — | all above | ⬜ |

---

## Design gaps to resolve (screenshot vs Gemini code)

| Gap | Decision owner | Notes |
|---|---|---|
| Top-bar right icons (notifications, avatar) not in CSS | Alexandra | Keep as static icons or drop for v1 (no accounts yet) |
| "My Feed" dropdown in screenshot | Alexandra | Needs accounts → v2; hide in v1 |
| Rank badges `#01–#03` vs `#4`, `#5` in screenshot | Alexandra | Pick one format (suggest zero-padded) |
| Community category has no tag color | Alexandra | Covered by C-02 |
| Per-item "SATIRICAL PARODY" label | Alexandra | Covered by W-06 |
| Unsplash placeholder images | Tornike | Replaced by real thumbnails from N-03 |
| "+14% Trend" needs history between runs | Tornike | Covered by C-03 / N-06 / N-10 |

## v2 parking lot (not in v1)

- User accounts + "My Feed"
- Custom voting / community ratings
- Creators page and Analysis page
- Analytics dashboards
- Newsletter / social auto-posting of Dispatch stories
