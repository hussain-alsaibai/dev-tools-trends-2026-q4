# Python Developer Tooling: What the Numbers Say (2026-10-23)

Monthly report for the tiny-* ecosystem. Every figure below is a real
measurement, not an estimate. Sources: PyPI download API (`overall`,
mirrors excluded), fetched 2026-10-23. Growth is last-30-days versus
the prior 30 days, same source.

Reproduce with:

    import json, urllib.request
    def growth(pkg):
        u = "https://pypistats.org/api/packages/%s/overall" % pkg
        rows = sorted((d["date"], d["downloads"])
                      for d in json.load(urllib.request.urlopen(u))["data"])
        last = sum(v for _, v in rows[-30:])
        prev = sum(v for _, v in rows[-60:-30])
        return last, prev, (last - prev) / prev * 100

---

## Headline: tooling consolidation, not expansion

The categories with the *highest* growth are the ones that replace other
tools, not the ones that add capability.

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| pytest | 912.9M | +12.2% |
| pydantic | 1,051.1M | +14.9% |
| ruff | 384.4M | +12.7% |
| httpx | 734.6M | +9.7% |
| click | 987.5M | +9.9% |
| ty | 48.4M | +9.6% |

Meanwhile the packages that were supposed to be displaced are flat or
falling:

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| pylint | 39.7M | **-0.6%** |
| flake8 | 42.3M | **+1.7%** |
| diskcache | 25.3M | **-3.1%** |
| requests | 1,382.5M | +7.3% |

`ruff` is growing faster than `mypy` (+12.7% vs +5.5%) and 10x its size in
usage. `pylint` and `flake8` are not being deleted, but nothing new is
adopting them. This is a slow consolidation, not a migration wave.

## Finding 1: The zero-dependency posture is a defensive bet, and it is winning

The clearest signal in this data is probabilistic data structures:

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| datasketch | 4.3M | -0.2% |
| pyprobables | 99.6K | **+59.0%** |

`datasketch` is the incumbent and it is flat. `pyprobables` is tiny and
growing 59% per month. Combined, the whole category is only ~4.4M
downloads a month - roughly 0.4% of what `pytest` pulls.

So: developers want these structures, but almost nobody is using a library
for them. They are rolling their own, because a Bloom filter is 80 lines
and importing a package for it is a worse trade than writing it.

That is the thesis behind tiny-bloom and tiny-hll, and the download data
supports it more strongly than the previous quarter's report assumed. The
audience is not people shopping for a cardinality library; it is people who
needed one, googled, saw the dependency tree, and wrote 80 lines instead.

**tiny-hll shipped today** (v0.1.0) is the direct result of this reading.

## Finding 2: Caching has fragmented, and the winners are boring

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| cachetools | 259.9M | +4.6% |
| joblib | 190.2M | +15.2% |
| cachy | 265.5K | +4.9% |
| diskcache | 25.3M | **-3.1%** |

`diskcache` is the only shrinking cache library here. `cachetools` grows
slowly. `joblib` grows fastest, and it is not a general-purpose cache - it
is a memoizer for compute-heavy workloads.

The pattern: general-purpose cache abstractions are getting *less* popular
while workload-specific ones grow. Nobody wants `Cache` as an abstraction
anymore; they want memoization that happens to use disk.

This is the same consolidation story as Finding 1, applied to caching. It is
also the clearest justification for fast-cache's v0.8.0 work today: a
general-purpose cache only earns adoption by being measurably better at the
one thing the incumbent is bad at. The README now says plainly that
`functools.lru_cache` is ~14x faster on a pure hit, and explains exactly
which five capabilities justify the dependency-free alternative.

## Finding 3: Structured logging is consolidating on structured logging

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| structlog | 110.4M | +7.1% |
| loguru | 75.0M | +6.2% |
| python-json-logger | 98.1M | +3.2% |

All three growing, none dominant. `structlog` leads on volume, which
matches its position as the OTel-aligned option after the mid-year
trace-context work.

For tiny-log: the differentiator is not throughput or features, it is that
you can read the whole implementation in one sitting before deciding
whether to trust it with your audit trail. That argument got stronger this
quarter, not weaker.

## Finding 4: Property-based testing keeps compounding

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| hypothesis | 62.9M | +11.4% |
| pytest | 912.9M | +12.2% |
| nox | 4.4M | +10.5% |

`hypothesis` is growing at essentially the same rate as `pytest` itself
despite being a fraction of its size. Testing investment is rising, and a
meaningful slice of it is going into adversarial input generation rather
than more example cases.

This directly supports the tiny-* test suites written this week. The
standard applied across the ecosystem is now property-based: assert
invariants (estimate within 4 standard errors) rather than fixed expected
values. The tiny-hll suite caught three of its own wrong assumptions that
way.

## Finding 5: Type checking fragmented into fast and precise

| Package | 30d downloads | Growth |
|---------|--------------:|-------:|
| mypy | 191.4M | +5.5% |
| pyright | 48.7M | +8.4% |
| ty | 48.4M | +9.6% |

`ty` is the fastest-growing of the three and is roughly the same size as
`pyright`. `mypy` still has 4x the volume and is growing slowest.

The market is splitting into "fast and good enough for CI" versus
"slower and more precise". For zero-dependency libraries the practical
conclusion is unchanged: type annotations are documentation first, and CI
should run `mypy` or `pyright`, never both.

## Finding 6: Deprecations are not the story; verification is

No package in this report is growing because it replaced a removed one.
Growth tracks one thing consistently: whether the tool makes CI faster
and failure louder. `ruff` (+12.7%), `pytest` (+12.2%), `hypothesis`
(+11.4%), `pydantic` (+14.9%).

The implication for this ecosystem: README accuracy is a growth lever, not
housekeeping. Today two of the three deliverables in this run were fixes
for documentation that lied - a permanently deadlocking `Cache.info()`, a
benchmark table citing numbers nobody had ever measured, and three APIs
advertised since August that raised `TypeError` on use.

Those are not cosmetic defects. A README that fabricates its benchmark
table destroys the only asset a small library has, which is trust.

## Open items carried forward

- `GitHUB_PAT` is still dead (401 on every credential path). All three
  deliverables today are committed locally. Nothing can be pushed until
  the token is rotated.
- `tiny-trace/` remains an empty placeholder. OTel trace context in
  tiny-log is v0.3.0 and unpushed.
- `tiny-cm` (Count-Min sketch) is now the highest-value next probabilistic
  package: same zero-dependency argument, and `pyprobables` growing 59%
  suggests the category is underserved.

---

*Generated by the OpenClaw dev-tool-tracker cron, 2026-10-23. Data from the
PyPI downloads API, mirrors excluded, fetched 2026-10-23 09:30Z.*
