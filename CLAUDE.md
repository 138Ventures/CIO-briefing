# Investor Behavioral Analysis — Project Handoff

## Owner
Markus Wolf — personal IBKR portfolio behavioral analysis tool

## Project Root
`/Users/markuswolf_mbpm2/Documents/10_AI/Claude/CoWork/Bhfs/YaInvest/`

## What This Project Does
Analyzes Markus's IBKR trading history to detect and quantify **emotional/behavioral investing deficits** using a 10-factor model. Outputs an interactive dashboard comparing his behavior to retail investors and top fund managers, with SGD cost quantification per factor.

---

## Architecture

### Data Pipeline
```
IBKR CSV exports  →  Python parsers  →  SQLite (portfolio.db)  →  Dashboard (HTML)
```

### Key Files

| File | Purpose | Status |
|------|---------|--------|
| `portfolio.db` | SQLite database — 19 tables, 624 trades, 60 months of returns | ✅ SOLID — do not rebuild |
| `build_portfolio_db.py` | Builds the database from all raw IBKR exports | ✅ Working |
| `behavioral_analysis_v4.py` | Computes 10 behavioral factor scores | ✅ Working, outputs JSON |
| `behavioral_report_v4.json` | Latest factor scores + analysis | ✅ Current |
| `dashboard-v2.html` | Interactive dashboard (Chart.js) | ❌ NEEDS CLEAN REBUILD — has rendering bugs from incremental edits |
| `dashboard.html` | Earlier version (simpler, more stable) | Reference only |
| `ibkr/portfolio-data-prep.plugin` | Claude plugin for parsing IBKR CSVs | ✅ Working |
| `ibkr/plugin_extracted/` | Extracted plugin contents | Reference |

### Raw Data Locations
```
ibkr/MULTI_20260313.csv                    — Latest daily activity statement
ibkr/transactions-sinceinception/          — Full trade history since 2021
ibkr/activity-reports-sinceinception/      — Period activity reports (overlapping)
ibkr/MTM summary/                          — Mark-to-market summaries by period
ibkr/Realized summary/                     — Realized P/L summaries by period
ibkr/Portfolio-Analyst-Sinceinception/     — PA report with monthly benchmarked returns
```

### Reference Files
```
Yainvest_IBI_report.pdf                    — Yainvest methodology reference (couldn't parse — needs poppler)
Yainvest Report Sample - ETF Fidelity...   — Sample Yainvest output
BhFS_Template_IBI_Portfolio Data (1).xlsx  — Yainvest input template (Transactions + Positions sheets)
```

---

## Database Schema (portfolio.db)

### Core Tables
- **trades** (624 rows): trade_date, account, symbol, quantity, price, proceeds, commission, cost_basis, realized_pl, mtm_pl
- **transactions** (664 rows): broader transaction log including deposits/withdrawals
- **position_snapshots** (215 rows): point-in-time position data with cost basis + unrealized P/L
- **instruments** (74 rows): symbol → description/category mapping

### Performance Tables
- **pa_monthly_performance** (60 rows): Monthly returns — account_return, spxtr_return, efa_return, vt_return (Feb 2021 → Feb 2026). **NOTE: March 2026 partial data (-10.52%) is NOT in this table yet — was manually added to dashboard only.**
- **pa_symbol_performance** (77 rows): Per-symbol contribution to return
- **pa_trade_summary** (75 rows): Per-symbol trade counts, commissions, realized P/L
- **nav_bridge** (6 rows): Annual NAV reconciliation with TWR

### Market Data
- **market_data** (2674 rows): Daily OHLCV for SPY, VT, and portfolio symbols
- **position_mtm** (656 rows): Period MTM performance per position

### Other
- **cash_balances**, **cash_flows**, **dividends**, **fx_rates**, **realized_pl**, **mtm_performance**

---

## 10-Factor Behavioral Model

Each factor scored 0–1 (0 = no deficit, 1 = severe). Benchmarked against avg retail investors and top-decile fund managers.

| # | Factor | Score | Severity | Cost (SGD) | Cost %/yr | Type |
|---|--------|-------|----------|-----------|-----------|------|
| 1 | Disposition Effect | 0.697 | MODERATE | -21,414 | -2.1% | Weakness |
| 2 | Concentration | 0.253 | LOW | 0 | 0% | Strength |
| 3 | Diversification (geo/ccy) | 1.000 | HIGH | -8,200 | -1.5% | Weakness |
| 4 | Volatility Reaction | 0.234 | LOW | +4,500 | +0.8% | Strength |
| 5 | Turnover | 0.806 | HIGH | -5,840 | -1.8% | Weakness |
| 6 | Performance Chasing | 0.673 | MODERATE | -33,968 | -4.2% | Weakness |
| 7 | Averaging Down | 0.420 | MODERATE | -14,426 | -1.4% | Weakness |
| 8 | Recency Bias | 1.000 | HIGH | -18,500 | -2.8% | Weakness |
| 9 | Overconfidence | 0.128 | MINIMAL | 0 | 0% | Strength |
| 10 | Correlation Neglect | 0.843 | HIGH | -12,370 | -2.3% | Weakness |

**Composite IBI: 0.605 (MODERATE)**
- Retail average: 0.545
- Average fund manager: 0.255
- Top decile fund manager: 0.133

### Benchmark Sources Used
- **Retail**: Odean 1998 (disposition), Barber & Odean 2000 (turnover), DALBAR 2023 (gap), Goetzmann & Kumar 2008 (diversification)
- **Fund managers**: S&P SPIVA reports, Morningstar Active/Passive Barometer
- **Top decile**: Renaissance, Bridgewater, AQR factor research

---

## Key Portfolio Facts

- **Two IBKR accounts**: U19937198 (SGD ~12K, SGX gold) + U7689636 (SGD ~223K, main)
- **Total deposited**: SGD 202,000
- **Current NAV** (Mar 19, 2026): SGD 208,055
- **Total absolute P/L**: SGD +8,721 (4.4% total over ~5 years)
- **TWR since inception**: approximately -40% (due to timing of deposits and drawdowns)
- **VT cumulative same period**: +72.6%
- **Behavioral gap**: SGD 136,092

### YTD 2026 (through Mar 19)
- You: **-5.95%** (Jan +7.06%, Feb -1.82%, Mar partial -10.52%)
- S&P 500: **-3.22%**
- VT: **-1.07%**

### Key Historical Periods
- **2021 (Feb-Apr)**: ~SGD 75K deployed into SMT, ARKK, PLTR, BILI, BIDU at peak euphoria
- **2022**: -65.9% TWR. Held all 2021 positions through massive drawdowns
- **2023**: +37.4% recovery. No chasing. Best behavioral year.
- **2024**: +30.5%. Good stock selection, disciplined sizing.
- **2025**: +20.8%. Turnover increased. Some chasing crept back.
- **2026 YTD**: -5.95%. Gold/commodity overweight hit by March correction.

### Current Position Concerns
- COPX: -23.6%, stop breached + chasing flag → EXIT HALF
- IBIT: -23.8%, 15% trailing breached → EXIT HALF
- HGT: -18.2%, 621-day loser, stop breached → EXIT
- SQN: -22.8%, both stops breached → EXIT
- BTGD: -38.6%, 15% trailing breached → EXIT
- AMZN: -16.9%, approaching 20% stop → WATCH

---

## Dashboard Rebuild Requirements

### What's Wrong With Current Dashboard
- 1,273-line single HTML file with incremental patches
- Charts fail to render due to canvas ID conflicts and lazy-init issues
- Tab switching doesn't consistently trigger chart initialization
- No module separation — all data, logic, and UI in one file

### Recommended Architecture for Rebuild
1. **Single HTML file is fine** (no server needed) BUT should use clean modular JS
2. Use Chart.js 4.x (already loaded via CDN)
3. **5 tabs**: Overview, 10 Factors, Cockpit, Drill-Down, Action Plan
4. Each tab's charts should init when tab is first shown (IntersectionObserver or tab click handler)
5. Period selector (SI, YTD, 2025, 2024, 2023, 2022, 2021) should update ALL charts below it
6. All performance charts must start at 0% on Jan 1 of the selected year

### Data Already Embedded in Dashboard
The dashboard-v2.html contains hardcoded JS arrays for:
- `MONTHLY` — 61 months of returns (you, SPX, VT) including Mar 2026 partial
- `FACTORS` — 10 factor objects with scores, benchmarks, costs, explanations
- `POSITIONS` — 21 current positions with flags, stop levels, actions
- `SAMEDAY` — 15 same-day round-trip trades
- `CHASING_YR` — chasing % by year
- `TRADE_VOL` — buys/sells count by year
- `DIV_ASSET/GEO/CCY` — diversification breakdown donuts
- `COUNTERFACTUAL` — trade category P/L impact

### Visual Design Spec
- Dark theme (black bg, dark cards)
- Font: Fira Sans + Fira Code (mono for numbers)
- Colors: orange (#f97316) = you, cyan (#06b6d4) = retail, purple (#a855f7) = fund mgr
- Red/green for losses/gains. Amber for warnings.
- Square legend markers (NOT circles — Chart.js pointStyle:'rect')
- Accessible: skip links, ARIA roles, tabular nums, sufficient contrast

---

## Suggested Next Steps (in fresh session)

### Immediate (fix what's broken)
1. Rebuild dashboard from scratch using clean architecture — extract data into a `data.js` section, rendering into `render.js` section
2. Test each tab independently before connecting
3. Use the `ui-ux-pro-max` or `frontend-design` skill for polish

### Short-term improvements
4. Add "What If" simulator — toggle rules on/off, see projected improvement
5. Monthly behavioral score tracking (trend line of IBI over time)
6. Move `portfolio.db` to a dedicated `/portfolio-ledger/` folder
7. Add `behavioral_scores` table for monthly IBI snapshots
8. Parse the Yainvest PDFs (need `brew install poppler` first)

### Medium-term
9. Position Entry Journal (pre-mortem prompts before each trade)
10. Alert system (cron job checking positions against stop rules)
11. Forward-looking cockpit with macro regime indicators (needs external API)

---

## Skills Available
- `ui-ux-pro-max` — UI design intelligence, 25 chart types, dark mode
- `frontend-design` — production-grade frontend interfaces
- `portfolio-data-prep` plugin — IBKR CSV parsing (in `ibkr/portfolio-data-prep.plugin`)
- `pdf` — for parsing Yainvest PDFs once poppler is installed
- `xlsx` — for reading/writing Excel files
- `superpowers:writing-plans` — for planning complex rebuilds
- `superpowers:dispatching-parallel-agents` — for parallel tab development

## Important Reminders
- IBKR CSV is multi-section flat file, NOT a standard table. Use `parse_ibkr.py`.
- Multi-account: U19937198 (small, SGX) + U7689636 (main). Track current account by watching Account Information rows.
- March 2026 data is partial (through Mar 19). PA monthly table only has Jan+Feb. March was added manually.
- German number formatting for LIQID data (period=thousands, comma=decimal) — not relevant for IBKR.
- The user's benchmark is **beating world equities (VT)**. All analysis should reference this.
- The user's worst behavioral pattern is **buying late after run-ups** (performance chasing + recency bias).
- The user keeps 1-share positions to "track" stocks — advised this is suboptimal (use watchlist instead).
