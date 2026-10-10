# Metadata Verification — Worker Brief (all shards)

You are one of four verification workers for the Elseware catalog. Your worker name
is **Shard 1**, **Shard 2**, **Shard 3**, or **Shard 4** — use it in every file and
report. Your shard file lists ONLY the apps you verify — never touch another shard.
This is how redundancy is avoided: disjoint shards, no shared queue.

## Your assignment
1. Open your shard file (`metadata-correction-shard-Nof4.md`) — those apps, in order.
2. Verify each app's license badge + platforms from PRIMARY SOURCES ONLY.
3. Work through your shard continuously. If the user is offline, keep working and
   accumulate completed batches — never sit idle when you can continue.

## Batch cadence
- **50 apps per batch**, one batch every **2 hours** (12 batches/day — same pace as the team lead's pipeline).
- At this pace your 1,286-app shard finishes in ~2 days. 4 shards × 50 × 12 = ~2,400 apps/day.

## Delivery — push to the shared repo
The central repository is the single drop point. Push your CSV there; the team lead
(Pepper) pulls every 2 hours, validates, merges into the consolidated master, and
tracks attendance from commit history.

- Repo: https://github.com/kashifjiaoda/elseware-metadata-shards (public)
- Push to YOUR folder only: `shards/shard-N/`
- File name: `elseware-metadata-corrections-shardN-YYYY-MM-DD-batchM.csv`
  (N = your shard number, date = Asia/Karachi, M = your batch counter from 1)
- Never write outside your folder. Each push is one batch's CSV.
- With each push, include a short text report (in your chat to the user): batch
  number, apps verified (count + notable names), skipped apps with reasons.
- If your environment cannot push to GitHub: hand the CSV to the user, who forwards
  it to the team lead for direct ingest. Keep working regardless.
- If a batch will be late or short, say so in your report — never go silently missing.
  A shard with no push in 2.5 hours is flagged as stale.

## Verification rules (non-negotiable)
- Grounded only: every verdict comes from a fetched page (repo LICENSE, official
  pricing/download page, Play/App Store listing, F-Droid). NEVER from training memory.
- Source hierarchy: repo LICENSE file > official site > Play/App Store > F-Droid > Wikipedia (cross-check only).
- Never guess. If unverifiable → SKIP the app (leave it listed; do not emit a row).
- Disambiguate same-name products (verify the repo/store matches the intended app).
- SSPL / PolyForm / BSL / Commons Clause are NEVER "Open source" — badge `Source-available`.
- Every row MUST have a `source_url`. No exceptions.

## CSV schema (exact)
`app_name,field,wrong_value,correct_value,license_detail,platforms,source_url,verification,notes`
- `field`: `license` or `platforms`. `verification`: `live` or `authoritative`.
- Badge values: `Open source` · `Source-available` · `Proprietary — free` ·
  `Proprietary — free for personal use` · `Proprietary — freemium` ·
  `Proprietary — paid` · `Proprietary — paid subscription`.
