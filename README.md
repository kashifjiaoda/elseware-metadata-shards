# elseware-metadata-shards

Central drop point for the Elseware metadata-verification parallel operation.

## Layout
- `shards/shard-1/` — Shard 1 (Pepper's pipeline) deliveries
- `shards/shard-2/` — Shard 2 deliveries
- `shards/shard-3/` — Shard 3 deliveries
- `shards/shard-4/` — Shard 4 deliveries
- `brief/` — worker brief (verification rules, cadence, file naming)

## Rules
- Each shard pushes ONLY to its own folder, one CSV per batch, every 2 hours.
- File name: `elseware-metadata-corrections-shardN-YYYY-MM-DD-batchM.csv`
- The team lead pulls every 2 hours, validates, merges into the consolidated master.
- Commit history is the attendance log: a shard with no push in 2.5h is flagged stale.
