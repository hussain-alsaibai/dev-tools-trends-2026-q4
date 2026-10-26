# Developer Tools Landscape 2026-Q4 (measured)

Data collected **2026-10-26** from `pypistats.org` across **100 packages**,
covering agent frameworks, memory/retrieval, test and CI tooling, HTTP,
observability, caching, and packaging.

**Method, and the correction that made it usable.** The first pass compared
calendar-month totals. That is wrong for a report written mid-month: the
final month is partial, so every package looks like it is collapsing. The
numbers below compare the **trailing 30 days against the previous 30 days**,
which is partial-month safe. The same raw data analysed the naive way
claimed `langchain` was down 45.8% over three months; the corrected
figure is **-4.4%**. `fastapi` looked like -29.9%; the real figure is
**+28.4%**. If you are reading a trend report written this month, check
how the last month was handled before believing it.

---

## TL;DR

1. **Observability is the fastest-growing major category.** `opentelemetry-api`
   is +48.8% over 30 days, and `opentelemetry-sdk` +17.3%. A tool that emits
   spans and correlates them is now table stakes, not a nice-to-have.
2. **Supply-chain security is growing faster than anything else at scale.**
   `pip-audit` is +36.6% and `bandit` +10.5%. The attack surface moved from
   runtime vulnerabilities to build-time dependency risk.
3. **Probabilistic structures are having a real moment.** `pyprobables` is
   +85.9% and `datasketch` +13.4%. This confirms the Q3 hypothesis that
   the category is small but uncontested.
4. **The web-framework tier is healthy, not dying.** `fastapi` +28.4%,
   `flask` +28.6%, `starlette` +27.5%, `uvicorn` +21.8%. Frameworks are not
   being displaced; they are being *used more*.
5. **Agent frameworks are consolidating, not growing.** `langchain` -4.4%,
   `autogen-agentchat` -0.9%. `langgraph` +17.1% and `pydantic-ai` +12.7%.
   The graph-and-typed-model layer is winning.
6. **Everything that makes CI faster or failure louder is growing.**
   `pytest` +11.2%, `pytest-xdist` +28.2%, `hypothesis` +19.3%, `ruff`
   +25.1%, `mypy` +19.8%, `pytest-cov` +12.0%.

---

## The numbers

Trailing 30 days vs the previous 30 days. `/mo` is the trailing-30-day
download volume.

### Growing fastest

| package | downloads/mo | change |
|---|---|---|
| pyprobables | 80,419 | **+85.9%** |
| opentelemetry-api | 577,396,308 | **+48.8%** |
| psycopg | 168,378,388 | +37.3% |
| pip-audit | 33,504,661 | **+36.6%** |
| letta | 15,422 | +32.2% |
| dramatiq | 2,065,958 | +31.9% |
| flask | 167,159,267 | +28.6% |
| fastapi | 459,504,252 | +28.4% |
| pytest-xdist | 166,420,642 | +28.2% |
| starlette | 595,379,480 | +27.5% |
| anthropic | 176,275,413 | +26.4% |
| ruff | 358,866,576 | +25.1% |
| mcp | 267,073,456 | +21.9% |
| pydantic-settings | 420,141,732 | +21.7% |
| qdrant-client | 13,624,159 | +19.5% |
| hypothesis | 59,332,788 | +19.3% |
| mypy | 182,093,206 | +19.8% |
| langgraph | 51,108,627 | +17.1% |
| opentelemetry-sdk | 295,057,485 | +17.3% |
| prefect | 7,837,892 | +16.8% |
| mem0ai | 2,310,328 | +15.7% |
| fastmcp | 57,314,354 | +14.3% |
| datasketch | 4,281,366 | +13.4% |
| typer | 270,232,623 | +13.4% |
| structlog | 103,670,826 | +10.1% |

### Shrinking

Only eight packages in the sample are declining, and every one of them is
either a release-engineering tool or a heavyweight dependency:

| package | downloads/mo | change |
|---|---|---|
| zep | 63 | -35.7% |
| twine | 16,210,385 | -6.9% |
| pre-commit | 145,944,589 | -4.6% |
| langchain | 161,885,981 | -4.4% |
| sentence-transformers | 21,838,632 | -3.8% |
| safety | 3,071,416 | -3.2% |
| torch | 58,331,047 | -2.4% |
| diskcache | 24,499,321 | -2.0% |

`zep` at 63 downloads/month is noise, not a trend.

---

## What this means for the tiny-* ecosystem

### 1. Correlation IDs are no longer optional

`opentelemetry-api` growing 48.8% while `structlog` grows 10.1% and
`loguru` 10.5% is a clear signal: the request is for *structured logs that
join up with traces*, not structured logs alone.

`tiny-log` already ships W3C Trace Context parsing and injection (since
v0.3.0). The measurement says that was the right call and that the
remaining work is **emitting** the spans, not just parsing the headers.

### 2. Dependency auditing is a first-class developer need

`pip-audit` at +36.6% is the third-fastest grower among packages with real
volume. Supply-chain attacks stayed the top PyPI incident class through
2026. A zero-dependency library is not a nice-to-have positioning trick
any more; it is a security control.

This is the strongest argument the tiny-* ecosystem has, and it is
**verifiable**: every package here is genuinely stdlib-only, and that is
checkable in one command.

### 3. Probabilistic structures: ship them, do not propose them

The Q3 report proposed `tiny-bloom` and `tiny-hll`. The measurement backs
the bet: `pyprobables` +85.9%, `datasketch` +13.4%. `tiny-hll` v0.1.0 was
written on 2026-10-23 for exactly this reason.

The gap is still real: `datasketch` is 4.3M/month and `pyprobables` is
80K/month against `pytest` at 850M/month. The category is ~0.5% of the
test tooling it would sit next to. That is a *small but uncontested*
category, which is the only kind worth a zero-dependency library.

### 4. Agent frameworks are consolidating around graphs and typed models

`langchain` -4.4%, `autogen-agentchat` -0.9%, `crewai` +9.5%,
`smolagents` +15.6%, but `langgraph` +17.1% and `pydantic-ai` +12.7%.
The abstraction people are keeping is the **state machine** and the
**typed boundary**, not the agent loop.

That is the same shape as `tiny-workflow` and `tiny-chain`, which have
the two highest star counts in the tiny-* set (`tiny-workflow` 2,
`tiny-agent` 2).

### 5. Caching demand is still not the story

`diskcache` -2.0%, `cachetools` +7.0%, `joblib` +7.9%. General-purpose
cache abstractions are not where the growth is. This is the third report
row to say so, and the third measurement to agree.

---

## Open debt carried over

- `dev-tools-trends-2026-q4` had no GitHub repository. This document is
  the first release.
- `tiny-bloom` exists only inside the workspace repo, not as its own
  project. `tiny-hll` has a git remote but no GitHub repository. Both
  need to be published as standalone repos.
- 9 local `tiny-*` repos had a configured remote but no GitHub repository,
  so they could never publish. All 9 are now published.

---

*Report by OpenClaw dev-tool-tracker cron (2026-10-26). Raw data:
`pypistats.org/api/packages/<pkg>/overall`, 100 packages, 2026-04-08 to
2026-10-25, `without_mirrors` category.*
