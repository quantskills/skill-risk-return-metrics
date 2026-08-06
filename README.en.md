# skill-risk-return-metrics

Single-symbol risk/return dossier skill: given a ticker and a date window, it computes in one call the annualized return, annualized volatility, Sharpe, Sortino, max drawdown, Calmar, win-rate and other allocator-side metrics, auto-routing A-share / HK / US by suffix.

> This first paragraph doubles as the platform summary (suggested Topics: `quant` `risk-metrics` `sharpe` `drawdown` `a-share` `hk` `us`).

> 中文说明见 [`README.md`](./README.md)（中文优先）。

## What it does

Give it a ticker + a date window; it pulls daily closes from the platform's own
`panda_data` (A-share) / `tqx_data` (HK / US) data layer and returns the standard
allocator ratios as a JSON string. Undefined ratios (flat series, zero drawdown)
come back as `null` instead of `NaN` or a crash.

Full spec — parameters, output schema, formulas, when-NOT-to-use — is in [`SKILL.md`](./SKILL.md).

## Layout

```
skill-risk-return-metrics/
├── SKILL.md                     # required: skill spec (usage, params, workflow, formulas)
├── README.md                    # Chinese-first readme (first paragraph = summary)
├── README.en.md                 # this file (English)
├── LICENSE                      # GPLv3
├── quantskills.yaml             # upstream manifest (license: GPL-3.0-only)
├── agents/                      # runtime adapters (cursor / hermes / openclaw)
│   ├── cursor-rule.mdc
│   ├── openai.yaml
│   └── portable-loader.md
├── scripts/
│   └── risk_return_metrics.py   # executable: async def run(...) -> str + CLI
└── references/
    └── example_output.md        # example output
```

## Run it

**As a platform skill** — the entry point is `async def run(...) -> str` in
`scripts/risk_return_metrics.py`, matching the Panda QuantFlow skill contract
(same shape as the shipped `analysis_technical` / `fx_rates` skills).

**Standalone CLI:**

```bash
python scripts/risk_return_metrics.py 600519.SH --lookback-days 250 --risk-free-rate 0.02
python scripts/risk_return_metrics.py 0700.HK --start-date 20260101 --end-date 20260601
```

Dependencies: `pandas`, `numpy` (metrics) + `panda_data` / `tqx_data` (data, injected by
the platform runtime). Without the data libs the skill returns a structured `Error: …`
string rather than raising.

## Status

Listed → Runnable → Verified. Local checks passing: `py_compile`, frontmatter lint
(lead ≤200 chars, `**Why:**` present), `run()` signature ↔ parameters match, and a
synthetic-data smoke test of every metric + edge case (illegal symbol, zero-volatility
series → `null` ratios).

## Disclaimer

- **Research & educational use only.** Output is informational research, **not investment advice**, and makes **no promise of returns** or profitability.
- **Data sources:** A-share via `panda_data`; HK / US via `tqx_data` (platform-provided). Output quality depends entirely on upstream data availability and correctness.
- **Assumptions & limitations:** metrics use daily closes, annualized with 252 trading days/year; ratios whose denominator is 0 (flat series, no drawdown/down-days) return `null`. Results are point-in-time; adjustments beyond what the upstream feed provides are not applied.
- **Risk boundary:** do not use as the sole basis for real-money decisions; validate independently and understand market risk.

## License

GPL-3.0-only. See [`LICENSE`](./LICENSE).
