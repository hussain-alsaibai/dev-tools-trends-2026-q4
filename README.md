# dev-tools-trends-2026-q4

**State of Developer Tooling 2026 — Q4, measured.**

Not a survey of blog posts. Every claim in the report is backed by
download data pulled from `pypistats.org` across 100 packages, and the
raw numbers ship with the report so you can check the arithmetic.

## What is in here

| file | what it is |
|---|---|
| `2026-10-28-developer-tools-landscape.md` | **the current report** -- 160 packages, measured 2026-10-28 |
| `2026-10-28-pyPI-numbers.md` | the raw measurements behind it, for all 160 packages |
| `2026-Q4-developer-tools-landscape.md` | the 2026-10-26 report -- 100 packages |
| `2026-10-26-pyPI-numbers.md` | its raw measurements |
| `2026-10-23-python-tooling-numbers.md` | the earlier Q3-close dataset |

## What the 2026-10-28 report says

Three findings, all backed by the raw table:

1. **The agent-framework story reversed.** `langchain` -3.4% and
   `llama-index` -14.9%, while `marvin` +80.1%, `agno` +29.8% and
   `langchain-anthropic` +29.3%. The meta-frameworks are flat-to-down and
   the usage is moving to small opinionated libraries and provider-specific
   packages. The Q3 report said "consolidating around graphs and typed
   models"; the wider sample says "fracturing around them".
2. **The database layer is outgrowing the framework layer.** `psycopg`
   +45.6% is the third-fastest growing package in the whole sample, ahead
   of `fastapi` +35.8% and `flask` +38.9%.
3. **Test tooling is growing in parallelism and timeouts, not assertions.**
   `pytest-xdist` +37.6% and `pytest-timeout` +30.5%, against `pytest`
   itself +16.7%. The scarce resource is CI wall-clock, not coverage.

Only 12 of 160 packages declined, and every one of them is a heavyweight
dependency, a legacy ORM/SDK, or a documentation generator -- not a category
in retreat.

## The one methodological point worth stealing

The first pass compared calendar-month totals and was wrong. The report was
written on the 26th, so the final month had 25 days instead of 31, and
every package looked like it was in freefall.

| package | naive calendar figure | corrected (30d vs 30d) |
|---|---|---|
| fastapi | +15.4% | **+35.8%** |
| pytest | +1.8% | **+16.7%** |
| ruff | +12.8% | **+34.5%** |
| opentelemetry-api | +31.2% | **+57.5%** |
| langchain | -15.8% | **-3.4%** |
| llama-index | -27.0% | **-14.9%** |

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
2026-04-08 to 2026-10-27, 160 packages.*
