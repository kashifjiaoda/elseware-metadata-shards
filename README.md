# elseware-metadata-shards

**If you are a Shard worker, this README is your complete instruction manual. You need nothing else — no separate brief, no extra onboarding.**

Give a new model three things and it can start: this repo URL, its shard number, and the repo token.

## The operation

Four workers ("Shard 1"–"Shard 4") verify license + platform metadata for 5,144 apps in
disjoint shards of 1,286 apps each. No app appears in two shards — redundancy is
impossible by construction. The team lead (Pepper, Shard 1) pulls this repo every
2 hours, validates every CSV, merges into the consolidated master, and runs
independent audit spot-checks.

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

## Step 2 — THE TWO-HOUR RULE (strict, no exceptions)

**A push MUST land in `shards/shard-N/` every 2 hours. No matter what happens.**

- Normal window: 50 apps verified → push the batch CSV.
- Short window (rate limits, blocks, outages): push the rows you DID verify, and
  report the shortfall to the coordinator. **Never invent rows to reach 50.**
- Zero verified (total outage): commit a `status-YYYY-MM-DD-HHMM.txt` in your folder
  explaining why. The commit itself is your heartbeat.
- Silence is the only failure. A shard with no commit in 2.5 hours is flagged
  **STALE** and the coordinator is alerted to nudge you.

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

### Known traps (real mistakes this pipeline has caught)

- Stacer → GPL-3.0 (not MIT) · Audacity → GPL-3.0 · Inkscape → GPL-2.0-or-later
- Bitwarden server → AGPL-3.0 · Element (web/mobile) → AGPL-3.0
- Waze → proprietary/free (not open source; only components were opened)
- "Open core" products: badge the main product, note the split in `license_detail`

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
# every 2 hours:
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

## Token safety

The repo token is scoped to this repository only (contents read+write). If it ever
leaks, it can be revoked without affecting anything else. Worst case, junk gets
pushed — validation rejects it before it ever reaches the master.
