# An 8,026-store index needs more than a country label

By Mathieu Colla, Aithos · 21 November 2026

In the US view of the Agent-Ready Index, 4,686 of 8,026 store records use at least some measurements collected from France. That is 58.4% of this snapshot. Calling it a survey of 8,026 US merchants would answer a question the dataset was never built to answer.

I work on this index. Here is how I would read its numbers before using them to choose an implementation priority.

## What the snapshot contains

The unit below is one unique store domain in the US-view export, captured on 21 November 2026. These are provenance fields, not failure rates.

| Property in the export | Store records | Share of 8,026 |
|---|---:|---:|
| Full measurement taken from France (`measured_from = fr`) | 1,574 | 19.6% |
| Some checks borrowed from France (`borrowed_fr` non-empty) | 3,112 | 38.8% |
| Either form of French measurement | 4,686 | 58.4% |
| No merchant country recorded | 2,383 | 29.7% |

The first two groups are disjoint in this snapshot. The missing-country group can overlap either one, so do not add all four rows.

The US view describes an intended observation location. It does not describe the merchant's country. Our US collection uses a data-centre VPN address. When that address cannot reach a store but the French measurement is available, the index can retain the French measurement and flag it. Some individual checks can also be filled from France. A refusal of our collection address is insufficient evidence that a store refuses a shopper's assistant.

## Three questions before quoting a percentage

**Who is in the denominator?** These 8,026 domains are the stores included in our export, not a random sample of world commerce. Stores whose homepages never answered are excluded by the index's inclusion rules. A percentage of these records must stay a percentage of these records.

**Where was the evidence collected?** Keep the merchant's country, the collection origin and any fallback separate. If you need a strictly US-observed cohort, filter the fallback fields explicitly and report what remains. Absence of a fallback flag still does not establish the merchant's location.

**When was each fact measured?** The export date is not a common audit date. Records carry `checked_at`; full French fallbacks also carry `measured_at`. Individual borrowed checks may be older than the enclosing US audit. A fresh export cannot make the underlying measurements fresh.

## Reproduce the provenance counts

This reads an existing Agent-Ready Index export. It does not visit stores, run diagnostics or initiate checkout. Save it as `provenance.py`, then run `python3 provenance.py shops.us.json` on a matching export.

```python
import json
import sys
from pathlib import Path

rows = json.loads(Path(sys.argv[1]).read_text())
assert len({r["domain"] for r in rows}) == len(rows)
n = len(rows)
assert n > 0
full = {r["domain"] for r in rows if r.get("measured_from") == "fr"}
partial = {r["domain"] for r in rows if r.get("borrowed_fr")}
for label, count in (
    ("Full French measurement", len(full)),
    ("Some French checks", len(partial)),
    ("Either", len(full | partial)),
    ("Missing merchant country", sum(not r.get("country") for r in rows)),
):
    print(f"{label}: {count}/{n} ({100 * count / n:.1f}%)")
```

The source export for these counts has SHA-256 `b9e0f9ac71ed333b245ec5e38c8addb3c28544ead4e3240e4e9828ab2c4e7971`. Later exports will produce different totals. This article publishes aggregates, not a copy of the store database.

## What this changes for a merchant

Before treating a score as an implementation backlog, read the observation origin and date on the underlying evidence. A network refusal, an absent document and a missing product field call for different investigations. Combining them into a single adoption headline loses the distinction you need to decide what to fix.

The index inspects public evidence; it does not exercise carts, checkout or payment. These counts establish neither protocol adoption across a country nor successful purchases by agents.

Source: [Agent-Ready Index ranking](https://agentready.aithos.app/ranking/?src=post.github), maintained by Aithos. I am publishing our own methodology and data, not an independent endorsement of the index.
