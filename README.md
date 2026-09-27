# ledger-roots

Public distribution of Intyga's anchored ledger checkpoint roots.

> **Status:** configured, but this is not evidence that production publication is live. An
> empty `roots.jsonl` means nothing has been published yet.

## Contents

`roots.jsonl` holds one JSON object per checkpoint, appended in sequence order. Each row has
exactly these eight fields:

| Field | Meaning |
| --- | --- |
| `seqStart`, `seqEnd` | Ledger sequence range covered by the checkpoint |
| `entryCount` | Number of entries in that range |
| `root` | Checkpoint Merkle root |
| `anchorRef` | Self/Rekor anchor identifier (fixed formats, never free text or URLs) |
| `anchoredAt` | Anchoring time |
| `prevChainHash` | `chainHash` of the previous row |
| `chainHash` | SHA-256 continuity hash (DEWP §5.4) over `prevChainHash`, `root`, the sequence range, `entryCount` and `anchoredAt` |

No events, identities, receipts, customer names or tenant IDs are published. Counts and
timestamps **do** reveal aggregate service activity.

The file is written hourly by an automated publisher in a separate private repository through
a dedicated GitHub App. Rows are only appended. Existing bytes are never rewritten, and every
change is a normal commit, so the history shows each addition.

## What this does and does not prove

- A row is published only after the database records witness evidence from **both**
  Rekor (`https://rekor.sigstore.dev`) and an RFC 3161 TSA (`https://freetsa.org`). The first
  checkpoint still waiting for evidence holds back every later row.
- **Recorded witness quorum is not independent cryptographic verification.** The publisher
  checks the database records and chain continuity. It does not re-verify Rekor signatures or
  timestamp tokens.
- `roots.jsonl` gives you roots and continuity. It is not the full anchor evidence and not a
  backup of that evidence. To verify a checkpoint, check its evidence bundle against Rekor and
  TSA trust that **you** pin independently. Never trust keys learned from the proof.
- A database owner could forge a new, internally consistent suffix. Publishing it here would
  not make it externally verified.
- GitHub is a distribution channel, **not WORM storage or an independent witness**. Repository
  administrators can still rewrite or delete history. Keep your own clones or snapshots.
- A quiet ledger produces no new rows. A missing new row does not prove an outage.

## Verifying

Fetch the file anonymously, recompute the `chainHash` chain from the first row and compare the
roots you rely on with an evidence bundle verified against your own trust configuration. If a
previously fetched copy is not a byte-for-byte prefix of the current file, treat that as a
serious incident.
