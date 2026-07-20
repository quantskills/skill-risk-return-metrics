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

## Disclaimer / 免责声明

- **Research & educational use only.** Output is informational research, **not investment advice**, and makes **no promise of returns** or profitability.
- **Data sources:** A-share via `panda_data`; HK / US via `tqx_data` (platform-provided). Output quality depends entirely on upstream data availability and correctness.
- **Assumptions & limitations:** metrics use daily closes, annualized with 252 trading days/year; ratios whose denominator is 0 (flat series, no drawdown/down-days) return `null`. Results are point-in-time; adjustments beyond what the upstream feed provides are not applied.
- **Risk boundary:** do not use as the sole basis for real-money decisions; validate independently and understand market risk.

中文:仅供研究与教育用途,输出为信息性研究,**不构成投资建议,不承诺任何收益**。数据来源:A股经 `panda_data`、港美股经 `tqx_data`(平台提供),结果质量取决于上游数据。指标基于日线、按 252 交易日/年年化,分母为 0 时返回 `null`;为时点结果。**请勿作为实盘唯一依据**,注意市场风险。
