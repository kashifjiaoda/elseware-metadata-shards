# elseware-metadata-shards

![Elseware logo](assets/elseware-logo.png)

**Verification data pipeline for [Elseware](https://elseware.directory)** — the software-alternatives directory, and its sister project [manuscriptOS.io](https://manuscriptos.io).

**If you are a Shard worker, this README is your complete instruction manual. You need nothing else — no separate brief, no extra onboarding.**

Give a new model three things and it can start: this repo URL, its shard number, and the repo token.

## The operation

Four workers ("Shard 1"–"Shard 4") verify license + platform metadata for 5,144 apps in
disjoint shards of 1,286 apps each. No app appears in two shards — redundancy is
impossible by construction. The team lead (Pepper, Shard 1) pulls this repo every
2 hours, validates every CSV, merges into the consolidated master, and runs
independent audit spot-checks.

## TASK BOARD — how you get new work (read this on every pull)

This repo is task-driven. New work appears here and in your shard folder — nowhere else.

**How to watch for new tasks:**
1. `git pull` at the start of every work session, and at least once every 2 hours.
2. Read this TASK BOARD section first. If your phase or assignment changed, it changed here.
3. Check your shard folder (`shards/shard-N/` or `expansion/shard-N/`) for new app lists.
4. Work ONLY what the task board marks ACTIVE. Anything marked STANDBY is not your work yet — starting it early creates rework.

**Current task board (updated by the team lead):**

| Phase | Status | Your action |
|---|---|---|
| Phase 1 — Tier-4 verification (5,144 apps, `shards/shard-N/`) | ACTIVE | Work your `apps.md` in order, batch by batch. |
| Phase 2 — Expansion verification (15,323 apps, `expansion/shard-N/`) | ACTIVE | Start NOW — both phases may run in parallel (separate folders, separate batch numbering, no collisions). |

**How to complete a task:** work your app list in order → verify per the playbook
below → write the batch CSV → push → the team lead audits and merges. When you push
your last batch of a phase, also push a `DONE-phaseN.txt` in your folder, then check
this task board for what's next.

## Step 0 — Know who you are

- Your shard number **N** (1–4, given by the coordinator).
- Your folder: `shards/shard-N/` — push ONLY here. Never write outside it.
- Your app list: `shards/shard-N/apps.md` — your 1,286 apps, in fixed order.
- Work the list in order. Batch **M** = apps ((M−1)×50+1) through (M×50). No separate
  progress file needed; the batch number encodes your position.

## Step 1 — The job

For each app, verify two things from primary sources and emit one CSV row per field:

- `field=license` → the correct license badge (see taxonomy below)
- `field=platforms` → the correct platform list (Windows, macOS, Linux, Android, iOS, Web)

## Step 2 — PACE AND HEARTBEAT

**Speed: as fast as quality allows.** Verify as many batches as you can without
compromising the result — max 50 apps per batch, push each batch when it is done.
The 2-hour window is a heartbeat, not a speed limit.

**Heartbeat (strict, no exceptions): a push MUST land in `shards/shard-N/`
at least every 2 hours. No matter what happens.**

- Batch finished → push the batch CSV.
- No batch finished in the window → commit a `status-YYYY-MM-DD-HHMM.txt` in your
  folder explaining where you are. The commit itself is your heartbeat.
- Short batch (rate limits, blocks, outages): push the rows you DID verify, and
  report the shortfall to the coordinator. **Never invent rows to reach 50.**
- Silence is the only failure. A shard with no commit in 2.5 hours is flagged
  **STALE** and the coordinator is alerted to nudge you.

**Rate limits:** if a source starts returning 429/403, back off — do not burn your
hourly budget chasing speed. Switch to other sources or apps and retry later.

## Step 3 — THE GROUNDING RULE (strict, most important)

**Every verdict must come from a page you fetched. NEVER from your training data.**

If you "know" an app's license from memory, that counts as NOT knowing it. Fetch the
page. Model memory is how the original catalog got poisoned with fabricated badges —
this operation exists to replace guesses with evidence.

## The verification playbook

### Source hierarchy (when sources conflict, higher wins)

1. Repo LICENSE file (raw `LICENSE` / `COPYING`)
2. Official site (pricing / download page)
3. Google Play Store / Apple App Store listing
4. F-Droid listing
5. Wikipedia — cross-check only, never the deciding source

### Case A — app claims to be open source

1. Find the canonical repo (GitHub / GitLab / Codeberg). Confirm it is the right
   product first (see Disambiguation).
2. GitHub: `GET /repos/{owner}/{repo}/license` → read `spdx_id`.
3. If NOASSERTION or no license endpoint: fetch the raw `LICENSE` or `COPYING` file
   and read it yourself. `package.json`'s `license` field is a hint, not evidence.
4. An F-Droid listing is strong FOSS evidence (F-Droid lists FOSS only). Debian/Fedora
   package metadata works as a cross-check.
5. Map SPDX to badge: MIT / Apache-2.0 / GPL / AGPL / BSD / MPL / EPL → `Open source`.
   Public-domain dedications (CC0-1.0, Unlicense, WTFPL) → `Open source` (more
   permissive than MIT; the taxonomy distinguishes restrictive-custom from truly-free).
   **SSPL / PolyForm / BSL / Commons Clause → `Source-available`. NEVER "Open source".**

### Case B — proprietary app (pricing tier)

1. Official site pricing or download page FIRST.
2. Mobile: Play Store listing (Free vs price shown, "in-app purchases" label) and
   App Store listing are authoritative pricing evidence.
3. Tiers: `Proprietary — free` (no paid tier exists) · `free for personal use` ·
   `freemium` (free tier + paid upgrades) · `paid` (one-time) · `paid subscription`.
   Put specifics (price, trial limits) in `license_detail`.

### Case C — platforms

Official download page. Play Store presence = Android; App Store = iOS
(Mac App Store = macOS). Wikipedia is a cross-check only.

### Disambiguation — same-name products (critical)

Names collide. **Verify the repo/store listing is the intended product BEFORE
emitting the row.** Check the repo description, linked website, and publisher.
Real failure from this pipeline: "Outline" — Jigsaw's Outline VPN (Apache-2.0) and
Outline wiki (BSL-1.1) are different products; the wrong repo was verified and the
row had to be scrapped.

### The no-guessing rule

Cannot verify from a primary source → **SKIP the app (emit no row).** A missing row
is honest; a guessed row is corruption. Skipped apps stay on the list for review.

### Badge taxonomy (exact values)

`Open source` · `Source-available` · `Proprietary — free` ·
`Proprietary — free for personal use` · `Proprietary — freemium` ·
`Proprietary — paid` · `Proprietary — paid subscription`

### The verification procedure (speed without breakage)

**Structure:** split every 50-app batch across 5 parallel verifiers, 10 apps each.
Serial verification is too slow — this is non-negotiable.

**Per app, 2–4 fetches maximum:**
- License: ONE fetch — the repo LICENSE file (open-source claim) or the official
  pricing page (everything else). Read the plan table + FAQ, never the CTA banner.
  Mobile app → App Store / Play Store listing (Free/Paid, IAP shown).
- Platforms: ONE or TWO fetches — official download page (best), or the store
  listings (App Store = iOS only, Play Store = Android only; claiming both needs both URLs).
- Identity: confirm the publisher / repo owner matches the app BEFORE verifying.

**3-minute rule:** if an app isn't verifiable in ~3 minutes, SKIP it with a reason
and move on. Never fight a bot-blocked site for 10 minutes. A correct skip beats a
slow guess.

**Grounded sources — check these, in this order:**

LICENSE:
1. Claims open source → the repo's LICENSE/COPYING file
   (raw.githubusercontent.com). License API as backup. Marketing pages never count.
2. Everything else → official pricing page (plan table + FAQ). Mobile → App Store /
   Play Store listing (Free/Paid, IAP shown).
3. Possibly dead → official shutdown post, archived-repo notice, or parked domain.
   Quote the evidence in notes.

PLATFORMS:
1. Official download page (lists everything at once).
2. App Store listing = iOS only. Play Store listing = Android only. Both claimed → both URLs.
3. Official help center ("we have no mobile app" is a valid source).

IDENTITY (do this first):
1. Store publisher name / repo owner must match the app.
2. Official domain must not redirect to something else (rebrand) or park (dead).

NEVER: aggregator download sites (soft112, uptodown, filehippo, softonic and
similar), reviews as your primary source, Wikipedia alone, or your training memory.

**Self-audit before every push:** exact 9-column schema, every row has a
`source_url`, no duplicate `(app_name, field)`, badges from the taxonomy only —
and re-check EVERY Open source / Source-available claim against the primary
source. One bad row = warning; a pattern of bad rows = escalation.

### Known traps (real mistakes this pipeline has caught)

- Stacer → GPL-3.0 (not MIT) · Audacity → GPL-3.0 · Inkscape → GPL-2.0-or-later
- Bitwarden server → AGPL-3.0 · Element (web/mobile) → AGPL-3.0
- Waze → proprietary/free (not open source; only components were opened)
- "Open core" products: badge the main product, note the split in `license_detail`
- Crypto-trap class: MetaMask → Source-available (ConsenSys proprietary license),
  Trezor Suite → Source-available (reference-source license), Uniswap/SushiSwap →
  Source-available (BSL-1.1). "Everyone knows it's open source" is wrong until the
  LICENSE file says so. New license variants appear — read what it actually says.
- Relicenses cut both ways: Morpho BUSL→GPL-2.0 (now open) · Braiins OS →
  proprietary dev-fee terms (no longer open; canonical repo deleted)
- Domain hijacks: verify the domain serves the real product (hop.exchange now serves
  a gambling site; parked domains redirect to /lander)
- Stale store text: App Store "open source" labels can predate a license change —
  verify against the repo's current LICENSE

## CSV schema (exact — rows that break this are rejected)

`app_name,field,wrong_value,correct_value,license_detail,platforms,source_url,verification,notes`

- `field`: `license` or `platforms`.
- `verification`: `live` (you fetched the source in this run) or `authoritative`
  (stable public fact with a cited URL).
- **Every row MUST have a `source_url`. No exceptions.**

## File naming

`elseware-metadata-corrections-shardN-YYYY-MM-DD-batchM.csv`

N = your shard number · date = Asia/Karachi (UTC+5) · M = your batch counter from 1.

## How to push

```bash
git clone https://<YOUR_TOKEN>@github.com/kashifjiaoda/elseware-metadata-shards.git
cd elseware-metadata-shards
# before each push:
git pull --rebase
# write your CSV to shards/shard-N/elseware-metadata-corrections-shardN-YYYY-MM-DD-batchM.csv
git add shards/shard-N/
git commit -m "shard-N batch M: <count> apps verified"
git push
```

Keep the token secret: never print it, never commit it, never paste it anywhere
except your git remote URL.

## What happens after you push

The team lead pulls every 2 hours, validates schema and sources, dedupes on
`(app_name, field)`, merges into the consolidated master, and re-verifies ~5% of
rows as an audit sample. Bad rows are rejected and reported back to the shard.
Commit history is the attendance log.

## Phase 2 — Expansion (15,323 apps) — ACTIVE

**Status: ACTIVE — owner approved 2026-10-11, START NOW.** Both phases may run in
parallel: `shards/shard-N/` (Phase 1) and `expansion/shard-N/` (Phase 2) are fully
separate — separate folders, separate batch numbering, no collisions. Push Phase-2
batches as you verify them; when you finish a phase's list, push `DONE-phaseN.txt`
in that phase's folder.

When ACTIVE, the rules:

- Your list: `expansion/shard-N/apps.md` — ~3,831 apps per shard (Shard 4: 3,830),
  balanced across tiers and categories. Ten apps per line.
- Your folder: `expansion/shard-N/` — push ONLY here. Never mix expansion files
  into `shards/shard-N/`.
- Batch numbering RESTARTS at 1:
  `expansion/shard-N/elseware-metadata-corrections-expansion-shardN-YYYY-MM-DD-batchM.csv`
- Batch M = lines 6+((M−1)×50) through 55+((M−1)×50) of your apps.md
  (50 lines = 50 apps, ONE app per line). Never split a line on "|" — some app
  names contain "|" characters; one line is always one app.
- Same schema, same grounding rule, same playbook, same heartbeat, same self-audit —
  everything in this README applies unchanged.
- Tier guidance: P0 = highest demand, verify first and fastest; P1 = regional apps,
  same rigor; P3 = developer tools, same rigor.
- At ~50 apps/batch this is ~77 batches per shard.

Shard 4 note: your expansion assignment is conditional — it activates only after you
clear the Phase-1 quality probation (3 clean batches under deep audit).

## Token safety

The repo token is scoped to this repository only (contents read+write). If it ever
leaks, it can be revoked without affecting anything else. Worst case, junk gets
pushed — validation rejects it before it ever reaches the master.

---

*Verification data pipeline for [Elseware](https://elseware.directory) — the software-alternatives directory — and its sister project [manuscriptOS.io](https://manuscriptos.io).*
