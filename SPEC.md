# carcamping.guide — Product Spec (v2 Vision)

_Source of truth for the next build phase. Drafted from Ian's blueprint. Current live version: v25.1._

---

## Overview

A community-built car-camping platform where every member keeps a **journal** of their adventures and contributions, and the whole community's activity rolls up into a **live master leaderboard**. Two dashboards share one design language; one **standard post** format powers every feed; a **map** ties all location content together; and a **trip planner** turns "planned" into a lasting itinerary record.

**Ethos:** transparency (contribute content and it's public) + community contribution ("so we can all share").

**Taglines:**
- Hero (main page): _"The best sights at the best sites."_
- Aspirational footer (light italic): _"The best things to see at the best places to be."_

---

## Problem Statement

Today the app has a broken/fragmented experience: a fabricated stat board, two overlapping "profile" views that confuse users, a feed that silently failed, and no structured content model. There's no clear reason for a visitor to sign up, and no easy first contribution for a new member. The vision fixes this with a single coherent content model, honest live metrics, and a low-barrier path to contribute.

---

## Goals

1. **One coherent model:** exactly two dashboards (public + personal), one standard post, no duplicate profile views.
2. **Honest, live metrics:** every leaderboard number is computed from real database content in real time — zero fabricated data.
3. **Low-barrier activation:** a new member can put a number on their scoreboard immediately (e.g., recommend one piece of gear) before ever camping.
4. **Content that compounds:** the more members post, the more valuable the public page becomes (social proof + fresh content).
5. **A real utility:** the trip planner/itinerary gives members a reason to return between trips.

---

## Non-Goals (this phase)

1. **User-vs-user ranked leaderboards / gamification** — the data model must _support_ it (keep counts per-user and per-category), but ranked leaderboards are a later phase.
2. **Reservations/booking integration** — bookings are logged manually, not booked through the app.
3. **Native mobile app** — responsive web only.
4. **Rich social-share link previews (OG cards)** — valuable, but a separate infra task (Netlify edge function).
5. **Messaging/DMs between members** — out of scope.

---

## The Two Dashboards

### 1. Main / Public Dashboard (the front door)

Deliberately minimal. Top to bottom:

- **Background image**
- **Intro copy** (hero)
- **Log in** (existing) + **Create account** (new)
- **Master leaderboard** (6 stats — see below)
- **Aggregate feed** immediately beneath the leaderboard — every member's posts, standard post format, newest first

### 2. Personal / Profile Dashboard (logged-in home = your profile)

**Logging in drops you onto your own profile page.** It is the _identical_ page anyone sees when they visit you — the only difference is you get **Edit** (and post) affordances; no one else does. There is no separate "dashboard that looks like a profile."

- **Identity:** photo · journal title · bio · contact · social handle
- **Personal scoreboard** (6 stats — see below)
- **Your feed** beneath it — your posts in sequence = your journal

---

## Master Leaderboard (Public) — 6 Stats

Each stat is **(a) a live number** that recalculates as content is posted, and **(b) an interactive link** that drills into that category's content.

| # | Stat | Counts (aggregate, all members) |
|---|------|--------------------------------|
| 1 | **Campsite Reviews** | total campsite-review posts |
| 2 | **Boondocking** | total boondocking-location posts |
| 3 | **Driving Tours** | total driving-tour posts |
| 4 | **Hot To-Dos** | total hot-to-do posts (all subcategories) |
| 5 | **Community Connect** | total members with a profile |
| 6 | **Gear Up** | total gear/equipment posts |

Four count **content**, one counts **people**, one is the **revenue** surface.

---

## Personal Scoreboard (Profile) — 6 Stats

Different set — this is _your journey + your contributions_.

| # | Stat | Source |
|---|------|--------|
| 1 | **Trips Planned** | from the Trip Planner (net-new feature) → feeds itinerary |
| 2 | **Nights Camped** | completed trips (a planned trip converts once its dates pass); record retained |
| 3 | **Campsite Reviews** | this member's campsite-review (and boondocking) contributions |
| 4 | **Driving Tours** | this member's driving-tour uploads |
| 5 | **Hot To-Dos** | this member's hot-to-do posts |
| 6 | **Hot Gear** | this member's gear posts (same category as "Gear Up", personal label) |

**Trips Planned + Nights Camped = the itinerary tracker** — a core value of the app.

> Open item: on the master board Boondocking is its own stat; on the personal board it's bundled under Campsite Reviews. Decide whether the personal board mirrors the master's separate content stats or keeps the bundle.

---

## The Standard Post (one format, everywhere)

Instagram/Facebook-style. Identical structure for **every** post; the _only_ branch is carousel vs. video. Rendered by one shared component in three places: the public aggregate feed, your journal, and anyone else's profile.

**Anatomy (top → bottom):**
1. **Category · Subcategory** (small eyebrow above the title)
2. **Lead media** — photo **carousel** _or_ **video** (most are carousels); lead image shows first
3. **Title**
4. **Body copy**
5. **Upvote count + upvote button**
6. **Author** — profile picture + link to their profile
7. (Gear only) **Buy / affiliate link**
8. (Location types) attaches a **GPS pin** to the map

Duplicates are allowed — multiple members can post the same spot; upvotes differentiate them.

---

## Content Taxonomy

Top-level content categories (= the four content leaderboard stats), plus Gear:

- **Campsite Reviews** — flat
- **Boondocking** — flat (location)
- **Driving Tours** — flat; **video** post with a GPS **start point** + optional **finish point**
- **Hot To-Dos** — has subcategories:
  - **Restaurants** — dining
  - **Shop** — shopping
  - **Activity** — hiking, trailheads, activities
  - **Hot Stops** — great views / day parking / day camping ("pull over, grab a photo, park a while — day-camp it if you're a car camper"). _Show an italic/parenthetical explainer._
- **Gear** — flat; recommend equipment; carries a buy/affiliate link (revenue)

Every **Hot To-Do** carries: GPS pin + carousel + write-up + upvote.

---

## The Map (central, unifying)

- One **searchable map of GPS pins**; all location content connects to it (campsite reviews, boondocking, hot to-dos, and driving-tour start/finish).
- **Filters:** Restaurant · Shop · Activity · Hot Stops (Hot To-Do subcategories), extensible to other content types.
- Built on the **existing Mapbox integration** (token already in the app; viewpoints/explore panels already use maps).
- **Driving tours** connect via start (+ optional finish) coordinates — representation on the map is an open design question.

---

## Trip Planner / Itinerary (net-new feature)

- A planner where a member schedules trips → each becomes a **Trip Planned** with an itinerary (nights, sites).
- When a planned trip's dates pass, it **converts to Nights Camped** and is retained as a record.
- Powers the top two personal-scoreboard stats and the "keep track of your adventures" value prop.
- Builds on the existing `trips` table (already fixed to persist, with `is_public`, `nights`, dates, `campground`, `site`).

---

## Current State / Foundation (what exists vs. net-new)

**Already built (reuse):**
- Auth, profiles (editable, photo upload/change fixed), full-page shareable profile with URL routing.
- `spots` table with `lat/lon/lng`, `photos[]`, `photo_url`, `tags`, `type`, `upvotes`, and latent `tour_start`/`tour_end` fields.
- `trips` table (persists correctly) + privacy-safe `shared_trips` view; itinerary bookings.
- A working social-post renderer (`renderFeedItem`) and a JS-join feed loader (spots→profiles has no FK, so we join in app code).
- Affiliate gear scaffolding (`GEAR` list, Amazon `carcampguide-20` tags).
- Mapbox integration.
- One-command deploy pipeline (git push → Netlify).

**Net-new (build):**
- Live 6-stat leaderboards (master + personal), fully computed — replaces fabricated `40`/`7` numbers.
- Photo **carousel** in the post (today only the first photo shows).
- **Video** posts (driving tours) — storage + player.
- **Map** browse/filter surface unifying all pinned content.
- **Trip planner** + planned→completed conversion.
- Hot To-Do **subcategories** + composer that routes media/pin by type.
- Restructure: login lands on profile-as-home; demote old dashboard tabs to a plain nav/toolbar.

**Known data cleanups:**
- Inconsistent `type` values (`boondock` vs `boondocking`); `status='approved'` vs `approved=false` mismatch. Standardize category values before scaling content.

---

## Requirements

### Must-Have (P0) — the coherent core
- Login lands on **profile-as-home**; eliminate the duplicate profile view.
- **Standard post** component (carousel + category/subcategory + title + body + upvote + author) shared across all feeds.
- **Live master leaderboard** (6 real counts) + **aggregate feed** on the main page.
- **Live personal scoreboard** (content stats) + journal feed on the profile.
- **Post composer** for all categories incl. Hot To-Do subcategories, with **pin drop** where applicable.
- **Photo carousel** support.
- Post-publish **auto-refresh** so new posts appear immediately.

### Nice-to-Have (P1) — fast follow
- **The map** browse/filter surface.
- **Video** posts for driving tours.
- **Trip planner** + Trips Planned / Nights Camped conversion.
- "Coming soon" cleanup + honest zero states.

### Future (P2) — design for, don't build yet
- **User-vs-user ranked leaderboards** (top contributors per category).
- Sponsorship/brand-partnership gear placements.
- Rich OG link previews (edge function).
- Driving-tour route visualization (start→finish line vs. two pins).

---

## Success Metrics

**Leading (days–weeks):**
- Signup conversion on the main page (visitors → accounts).
- % of new members who make ≥1 post within 7 days (gear = the easy first post).
- Posts created per week; leaderboard totals trending up.

**Lagging (weeks–months):**
- Member retention / return visits (trip planner as the hook).
- Gear affiliate click-through and revenue.
- Growth in map-pinned content density.

---

## Open Questions
- **Design:** how do driving-tour start/finish points render on the map (two pins vs. a route line)?
- **Product:** does the personal scoreboard mirror the master's separate Boondocking stat, or bundle it under Campsite Reviews?
- **Product:** do Hot Views survive (they're dropped from the leaderboard) — merge into Hot To-Dos or retire?
- **Eng:** video hosting — Supabase storage + native `<video>`, or an external host? Size/transcoding limits?
- **Data:** standardize category `type` values and the approval model before content scales.

---

## Suggested Build Order (phasing)

1. **Phase 1 — Structure & honesty:** profile-as-home restructure; standard post + carousel; live 6-stat master + personal boards; aggregate + journal feeds; auto-refresh. (Mostly P0; reuses existing pieces.)
2. **Phase 2 — Map & media:** map browse/filter over pinned content; video posts for driving tours; Hot To-Do subcategory composer.
3. **Phase 3 — Itinerary:** trip planner + planned→completed conversion.
4. **Phase 4 — Monetize & rank:** gear/sponsorship surfaces; per-user leaderboards; OG previews.

---

## Feed & Native Ads (design detail)

**One mobile-first stream, everywhere.** Design for a single column — most traffic is phones. No parallel/second stream (needs horizontal room phones don't have). The same standard-post card powers all three surfaces:
- **Main page:** the master aggregate feed (everyone's posts, sits beneath the leaderboard).
- **Profile page:** that member's posts in sequence = their journal.
- (Both already run on `renderFeedItem`.)

**Filter chips + sort** at the top of the feed:
- Chips: **All · Campsites · Boondocking · Driving Tours · Hot To-Dos · Gear**. Tapping "Gear" gives a gear-only stream on demand — this is how "gear as its own part of the feed" is satisfied without a second column.
- Sort: **Latest** and **Top** (by upvotes). "Top" is where community ranking pays off — best content rises — and it doubles as the SEO quality signal later.

**Native ads = featured product posts, not banners.** Gear is already a post type, so an advertiser is simply a member whose gear post is **featured/sponsored**:
- Reuses the exact post infrastructure (photo/carousel, write-up, buy link) — **no separate ad system to build**.
- Interspersed in the single stream at a cadence (~1 sponsored per 6–8 organic posts), each clearly labeled **"Sponsored"** / **"Featured Gear"** with affiliate disclosure.
- Native product listings convert far better than banners and keep the authentic community aesthetic.
- Implementation hook: add a `sponsored` (bool) + optional `sponsor` field on gear posts; feed injects them on a cadence; they also appear under the "Gear" chip.

**Trip-Recap post — its own rich card type.** Auto-composable after a completed trip (ties to the trip planner): hero = map route *or* a photo collage; headline ("4 nights on the Big Sur coast"); a stat strip (**nights · spots · miles**); a photo carousel; tappable links to the spots visited. Lives in both the feed and the journal, and is the single most shareable unit on the platform (pairs with the OG-preview work).

**Everything is authentic-first:** founder content, member content, and sponsored content all use the same card; the only differences are the "Sponsored" label and author attribution.
