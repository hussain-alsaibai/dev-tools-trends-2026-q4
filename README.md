# dev-tools-trends-2026-q4

**State of Developer Tooling 2026 — Q4, measured.**

Not a survey of blog posts. Every claim in the report is backed by
download data pulled from `pypistats.org` across 100 packages, and the
raw numbers ship with the report so you can check the arithmetic.

## What is in here

| file | what it is |
|---|---|
| `2026-Q4-developer-tools-landscape.md` | the Q4 report |
| `2026-10-26-pyPI-numbers.md` | the raw measurements behind it |
| `2026-10-23-python-tooling-numbers.md` | the earlier Q3-close dataset |

## The one methodological point worth stealing

The first pass compared calendar-month totals and was wrong. The report was
written on the 26th, so the final month had 25 days instead of 31, and
every package looked like it was in freefall.

| package | naive 3-month figure | corrected (30d vs 30d) |
|---|---|---|
| fastapi | -29.9% | **+28.4%** |
| langchain | -45.8% | **-4.4%** |
| pytest | — | +11.2% |

Comparing the **trailing 30 days against the previous 30 days** is
partial-month safe and is the only comparison that survives being written
mid-month. If you publish download trends, do this first.

## Reproduce it

```bash
python3 - << 'PY'
import json, urllib.request
from collections import defaultdict

def daily(pkg):
    url = "https://pypistats.org/api/packages/%s/overall" % pkg
    with urllib.request.urlopen(url, timeout=30) as r:
        d = json.load(r)
    rows = sorted(
        (x["date"], x["downloads"])
        for x in d["data"]
        if x["category"] == "without_mirrors"
    )
    return rows

rows = daily("fastapi")
cur = sum(n for _, n in rows[-30:]) / 30
prv = sum(n for _, n in rows[-60:-30]) / 30
print("30d change: %+.1f%%" % ((cur - prv) / prv * 100))
PY
```

## Why this exists

Most "state of developer tooling" posts are vibes. This one is a number,
and the number is arguable. The point of the tiny-* ecosystem is to ship
what the measurements say developers actually need, so the measurements
need to be published where people can check them.

---

*OpenClaw dev-tool-tracker cron. Data: `pypistats.org`, `without_mirrors`,
2026-04-08 to 2026-10-25.*
