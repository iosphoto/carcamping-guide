# HANDOFF — carcamping.guide (for Claude Code)

Single-file web app (`index.html`, ~3,800 lines). Supabase backend, Mapbox, deployed to Netlify.
**Read `SPEC.md` in this repo for the full product vision.** This file is the practical build state.

---

## FIRST ACTION

Nothing pending. **Live version is v29** — all committed and pushed. Verify the latest deploy rendered (Netlify auto-builds on push, ~1 min), then pick up the next task below.

**Recent history:**
- **v27:** "⛺ Campsite Review" category in the "+ Post" composer (routes through `submitPost` → `spots`), plus post-publish auto-refresh (feed + landing feed + leaderboard + open profile update, no reload).
- **v28:** Swipeable photo **carousel** in the standard post (`renderFeedItem`) — CSS scroll-snap track, hover arrows (hidden on touch), dots, and an "N/M" counter. Combines `photo_url` + `photos[]` and dedupes, because the two posting flows stored `photos[]` inconsistently (one includes the lead image, one doesn't).
- **v29:** Retired the broken "Share a favorite site" form. It wrote to a `site_reviews` table that never existed. Campground cards' button now routes to the standard `+ Post` composer pre-set to **campsite-review** with the campground name prefilled (`openPostForm(cat, prefillLoc)`). Removed the old submit form, the "Favorite sites" reader, and all associated JS/CSS. One way to post.

> **Note:** no JS runtime (node/deno/bun) is available in the Claude Code shell here, so the `node --check` sanity step can't run. Fallback used: a Python script extracts the inline `<script>` block and checks bracket/backtick balance. Cowork should do the real browser verification.

---

## Deploy pipeline (already set up)

- Repo: `github.com/iosphoto/carcamping-guide`, branch `main` → Netlify auto-deploys on push.
- A fine-grained PAT is stored in the macOS keychain, so `git push` needs no prompt.
- **Deploy = one command:** `git add -A && git commit -m "…" && git push`.

## Versioning convention (follow it)

1. Update the version comment at the very top of `index.html`: `<!-- vN · short description -->`.
2. After editing `index.html`, copy it to a snapshot: `cp index.html carcamping-vN.html` (must be byte-identical).
3. Sanity-check the inline script compiles before pushing (extract the big `<script>` block, `node --check`).

---

## Architecture / key facts

- **Supabase project ref:** `mjdgnpxbicnzmbinvqdy`. Anon key (JWT `eyJ…`) is embedded in `index.html` as `SUPA_KEY`. Admin email: `iosphoto@me.com`.
- **Tables:**
  - `profiles` — user_id, display_name, bio, zip, instagram, website, avatar_url, heading. RLS on; owner can CRUD own; anyone can read.
  - `spots` — the universal post table. Fields incl: name, description, type, region, tags[], photo_url, photos[], lat, lng, upvotes, status, user_id, tour_start, tour_end, gear_brand/price/url. **RLS is OFF** (anyone reads all).
  - `trips` — campground, site, start_date, end_date, rate, conf_number, nights, region, site_detail, is_public, share_slug. Owner-scoped RLS policies added.
  - `shared_trips` (view) — public-safe subset of trips (no rate / conf_number). Anyone can read.
  - `upvotes` — spot_id, user_id.
- **No foreign key** from `spots.user_id` or `trips.user_id` → `profiles`. PostgREST embedding (`select('*,profiles(…)')`) FAILS. **Always join authors in JS** — use the `attachAuthors(spots)` helper.
- **Posting:** "+ Post" → `selectPostCat` → `submitPost` inserts into `spots`. Categories: campsite-review, hot-todo, hot-view, driving-tour, boondocking, gear. Admin (`iosphoto@me.com`) auto-approves (`status='approved'`); everyone else → `status='pending'`.
- **Profile page:** `openPubProfile(userId)` — full page, deep-linkable via `?profile=<id>`, identical for owner and public (owner just gets Edit). Login should eventually land here (see SPEC).
- **Leaderboard:** `loadPlatformStats()` counts `spots` by type + `profiles` count. 6 live stats: Campsite Reviews, Boondocking, Driving Tours, Hot To-Dos, Community Connect (members), Gear Up.
- **Feeds:** `renderFeedItem()` is the shared social-card renderer; `loadMainFeed()` (logged-in) and `loadLandingFeed()` (landing aggregate).
- **Storage:** `trip-photos` bucket (public). Avatars use **unique filenames** (INSERT allowed, overwrite blocked by RLS) — don't reintroduce fixed-path `upsert:true`.

---

## Next tasks (priority order)

1. ✅ **Photo carousel** — done in v28.
2. ✅ **Retire the broken "Share a favorite site" form** — done in v29.
3. **Profile-as-home restructure** (SPEC P0) — login should land on your own profile page; demote the old dashboard tabs to a plain nav/toolbar; eliminate the duplicate profile view. Note the profile **journal** still uses a legacy renderer (`feed-card` / `feed_type`, ~line 2760) with its own photo/compass markup — migrating it to the shared standard-post renderer (`renderFeedItem`, now with the carousel) is part of this task.
4. From `SPEC.md` (P1+): video posts for driving tours, the map browse/filter surface, Hot To-Do subcategories (Restaurants/Shop/Activity/Hot Stops), trip planner (Trips Planned → Nights Camped), per-user leaderboards.

## Gotchas

- Supabase key must be the **JWT anon key** (`eyJhbGc…`), NOT `sb_publishable_…`.
- **Never delete/recreate the auth user** `iosphoto@me.com` (UID `8afbf869-2ce4-44c3-afa5-5b29d0ceb960`) — it would orphan data.
- Escape apostrophes in JS strings; never declare the same `const` twice.
- **Data inconsistency to clean up:** `spots.type` values aren't standardized (`boondocking` vs `boondock`), and `status='approved'` vs the `approved` boolean don't always agree. Code filters on `status='approved'`. Standardize before scaling content.
- Keep `index.html` and the `carcamping-vN.html` snapshot identical.

## Division of labor with Cowork (the desktop assistant)

- **Claude Code (you):** the code + git build loop — edit, commit, push.
- **Cowork:** live browser verification & screenshots, Supabase dashboard/admin/migrations, visual QA. Loop it back in to confirm a deploy renders correctly or to run anything against Supabase.
- Only one editor of `index.html` at a time; whoever edits, commit+push before the other starts.
