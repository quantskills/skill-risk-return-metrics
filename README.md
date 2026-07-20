# skill-risk-return-metrics

单标的风险收益画像 skill：给一个股票代码和时间窗口，一次算出年化收益、年化波动率、夏普、索提诺、最大回撤、卡玛比率、胜率等配置端常用指标，自动按代码后缀路由 A股 / 港股 / 美股。A one-call risk/return dossier for a single symbol across A-share / HK / US markets.

> 首段即平台简介（Topics 建议：`quant` `risk-metrics` `sharpe` `drawdown` `a-share` `hk` `us`）。

## What it does

Give it a ticker + a date window; it pulls daily closes from the platform's own
`panda_data` (A 股) / `tqx_data` (港股 / 美股) data layer and returns the standard
allocator ratios as a JSON string. Undefined ratios (flat series, zero drawdown)
come back as `null` instead of `NaN` or a crash.

Full spec — parameters, output schema, formulas, when-NOT-to-use — is in [`SKILL.md`](./SKILL.md).

## Layout

```
skill-risk-return-metrics/
├── SKILL.md                     # 必须：技能说明（用途、参数、工作流、公式）
├── README.md                    # 本文件（首段=简介）
├── scripts/
│   └── risk_return_metrics.py   # 可执行代码：async def run(...) -> str + CLI
└── references/
    └── example_output.md        # 示例输出
```

## Run it

**As a platform skill** — the entry point is `async def run(...) -> str` in
`scripts/risk_return_metrics.py`, matching the Panda QuantFlow skill contract
(same shape as the shipped `analysis_technical` / `fx_rates` skills).

**Standalone CLI** (for the community "Runnable" check):

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
