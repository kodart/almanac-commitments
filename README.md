# Almanac forecast commitments

This repository holds one file, `commitments.log`. Each line is a commitment to a private,
append-only log of probability forecasts:

```
<UTC time> seq=<rows in the log> head=<SHA-256>
```

The forecasts themselves are not here. A line reveals nothing about them; it only fixes them.

## What a line proves

Every forecast is stored as one row of canonical JSON (`payload`) and chained to the row
before it:

```
row_hash = sha256(prev_hash + "\n" + payload)        # prev_hash of the first row is 64 zeros
```

`head` is the `row_hash` of row `seq`. Changing, removing or inserting any earlier row changes
every hash after it, so a line fixes all rows 1…seq. The date a line was pushed here (see the
commit history, as recorded by GitHub) is therefore a date by which all of those forecasts
already existed, with their probabilities, their forecast times and a hash of the evidence
they were made from.

It proves nothing earlier than that push date. Rows marked `"src":"backfill"` in their
payload were made before this log existed and are covered only from the first line onward.

## How to check, once rows are published

Given the exported rows (JSON lines with `seq`, `payload`, `prev_hash`, `row_hash`):

```python
import hashlib, json
prev = "0" * 64
for line in open("rows.jsonl"):
    r = json.loads(line)
    assert r["prev_hash"] == prev
    prev = hashlib.sha256((prev + "\n" + r["payload"]).encode()).hexdigest()
    assert r["row_hash"] == prev
    # prev is now the head for seq == r["seq"]: compare with the line in commitments.log
```

Lines are appended once a day when new forecasts exist and are never edited or removed; the
git history of this repository is the record of that.
