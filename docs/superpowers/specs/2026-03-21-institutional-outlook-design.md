# Monthly Institutional Investment Outlook — Design Spec

**Owner:** Markus Wolf
**Date:** 2026-03-21
**Status:** Draft

---

## Purpose

A monthly, repeatable workflow that sources publicly available research from top-tier institutional investment firms, synthesizes it into a strategy memo with structured data, and maps the findings against Markus's current IBKR portfolio and 10-factor behavioral model. The goal is to stay current on the best institutional thinking, surface consensus and divergence, and generate accountability for portfolio decisions.

---

## Output

Two files per month:

- `outlooks/outlook-YYYY-MM.md` — strategy memo (5-8 pages)
- `outlooks/outlook-YYYY-MM.json` — structured companion data for diffing and dashboarding

---

## Source Universe

### Tier 1 — Always Include

| Source | Type | Extract |
|--------|------|---------|
| BlackRock Investment Institute | Weekly/monthly commentary | Asset class views, regime call |
| Vanguard Economic & Market Outlook | Quarterly + updates | Return forecasts, allocation shifts |
| PIMCO Cyclical/Secular Outlook | Quarterly | Rates, credit, duration positioning |
| JPM Asset Mgmt Guide to the Markets | Quarterly (70+ charts) | Valuations, earnings, macro data |
| Goldman Sachs Research (public) | Monthly macro outlook | Growth forecasts, S&P targets |
| Morgan Stanley (public notes) | Weekly/monthly | Equity strategy, cycle positioning |
| BofA Global Research (public) | Monthly | Fund manager survey, flow data |
| Howard Marks (Oaktree) memos | Irregular (~monthly) | Risk appetite, market psychology |
| GMO quarterly letters | Quarterly | 7-year asset class forecasts |
| AQR Insights | Irregular | Factor timing, alternative risk premia |

### Tier 2 — Include When Available

| Source | Type | Extract |
|--------|------|---------|
| Bridgewater Daily Observations | Excerpts via media | Macro regime, deleveraging signals |
| Hussman Weekly Market Comment | Weekly | Valuation + internals framework |
| Lyn Alden (public newsletter) | Monthly | Fiscal dominance, liquidity cycle |
| Luke Gromen FFTT | Public excerpts/interviews | Dollar system, energy/gold thesis |
| Druckenmiller / major PM interviews | Bloomberg, CNBC, conferences | High-conviction directional calls |
| BIS Quarterly Review | Quarterly | Global financial stability, cross-border flows |
| Fed dot plot / ECB forward guidance | Post-meeting | Rate path, QT pace |

### Tier 3 — Data Anchors

| Source | Data |
|--------|------|
| FRED / Fed | Rates, CPI, unemployment, yield curve |
| CME FedWatch | Market-implied rate probabilities |
| AAII sentiment survey | Retail sentiment gauge |
| CNN Fear & Greed Index | Multi-factor sentiment composite |
| BofA Fund Manager Survey | Crowding, cash levels, tail risks (via media summaries) |

### Sourcing Method

- **Direct web fetch (Firecrawl):** BlackRock, Vanguard, PIMCO, JPM, Goldman, AQR, Oaktree, GMO, Hussman, Lyn Alden insight pages
- **Media/interview extraction:** Druckenmiller, Dalio, Gromen via Bloomberg/CNBC/Reuters/podcast summaries
- **Data endpoints:** FRED API, CME FedWatch, AAII, CNN Fear & Greed

### Constraints

- Only publicly available content — no paywalled research
- Some firms publish with 1-2 week lag (GMO, PIMCO quarterlies) — noted when a view is from a prior period
- BofA Fund Manager Survey sourced via financial media summaries (full report is paywalled)
- Copyright: synthesize and cite, never reproduce large blocks. All views attributed.

### Failure Modes

- **Minimum threshold:** At least 6 of 10 Tier 1 sources must be successfully fetched for the report to proceed. If fewer, abort and notify.
- **Tier 2/3 failures:** Non-blocking. Note in `meta.sources_consulted` vs `meta.sources_available`.
- **Stale data:** If a source's latest content is >45 days old, carry forward the prior view in the consensus matrix with a staleness flag: `"stale": true, "as_of": "2026-01"`. Do NOT treat stale views as current consensus.
- **Fetch blocked:** If Firecrawl is blocked by a source, attempt media/secondary sources. If still unavailable, mark as `null` in the matrix with a note in the report.
- **Schema changes:** If a source restructures its page and extraction fails, skip that source and flag for manual URL update in `outlook-sources.json`.

---

## Report Structure (outlook-YYYY-MM.md)

### Section 1: Executive Summary (~200 words)
- 3-5 bullet "what changed this month"
- One-line regime call: where are we in the cycle?
- Overall risk appetite reading (risk-on / neutral / risk-off)

### Section 2: Macro Regime (~400 words)
- Growth trajectory (acceleration / deceleration / recession)
- Inflation path (sticky, falling, re-accelerating)
- Central bank stance (Fed, ECB, BOJ — hawkish/dovish shift?)
- Liquidity conditions (QT pace, reserve levels, credit impulse)
- Key data that moved the needle this month

### Section 3: Asset Class Views (~500 words)
- US equities, international DM, EM equities
- Government bonds / duration
- Credit (IG, HY)
- Commodities (energy, industrial metals, gold)
- Crypto / digital assets
- Cash & money markets
- For each: consensus direction + where the smart disagreements are

### Section 4: Consensus Matrix (table)

```
| Asset Class     | BLK | VGD | PIMCO | JPM | GS | MS | BofA | Marks | GMO | Consensus |
|-----------------|-----|-----|-------|-----|----|----|------|-------|-----|-----------|
| US Equities     | OW  | N   | UW    | OW  | OW | UW | N    | —     | UW  | SPLIT     |
| Intl DM         | OW  | OW  | N     | OW  | N  | OW | OW   | —     | OW  | BULLISH   |
| EM Equities     |     |     |       |     |    |    |      |       |     |           |
| Govt Bonds      |     |     |       |     |    |    |      |       |     |           |
| Credit (IG/HY)  |     |     |       |     |    |    |      |       |     |           |
| Commodities     |     |     |       |     |    |    |      |       |     |           |
| Gold            |     |     |       |     |    |    |      |       |     |           |
| Crypto          |     |     |       |     |    |    |      |       |     |           |
| Cash            |     |     |       |     |    |    |      |       |     |           |
```

Legend: OW = overweight, N = neutral, UW = underweight, — = no stated view
Consensus derivation rules:
- **BULLISH:** >60% of firms with a stated view say OW
- **BEARISH:** >60% of firms with a stated view say UW
- **SPLIT:** neither OW nor UW exceeds 60%
- **SHIFTING:** consensus flipped direction vs. prior month (requires prior month JSON)

### Section 5: Thematic Spotlight (~300 words)
- 1-2 high-conviction themes that multiple firms are converging on
- Include the contrarian counter-argument

### Section 6: Risk Radar (~200 words)
- Top 3-5 tail risks being flagged across research
- Probability-weighted where possible (using BofA FMS data)
- What would change the base case

### Section 7: Portfolio Alignment (~300 words)
- Map current positions against the consensus matrix
- Flag: where you're WITH consensus, AGAINST consensus, or have NO exposure to a consensus overweight
- Call out largest position-level mismatches

### Section 8: Behavioral Check (~200 words)
- Cross-reference current positioning against the 10-factor behavioral model
- Flag if any current position or planned trade shows a behavioral pattern
- Tie back to IBI score trends

### Section 9: Action Implications (~300 words)
- Per-position action implications where consensus, risk rules, and behavioral signals converge
- Format per position:
  - **POSITION:** ticker (current P/L%)
  - **CONSENSUS:** institutional view on the asset class (X of Y firms)
  - **BEHAVIORAL FLAG:** which factors are active
  - **IMPLICATION:** reduce / hold / add / watch
  - **CONVICTION:** HIGH / MODERATE / LOW — based on alignment of consensus + stops + behavioral flags
  - **COUNTER-CASE:** who disagrees and why
- HIGH conviction = consensus + your risk rules + behavioral model all point the same way
- This section creates accountability — harder to ignore a stop breach when multiple signals converge
- NOT trade recommendations. These are "institutional consensus implications" — the decision remains with you.

---

## JSON Schema (outlook-YYYY-MM.json)

```json
{
  "meta": {
    "month": "2026-03",
    "generated": "2026-03-21",
    "schema_version": "1.0",
    "sources_consulted": 14,
    "sources_available": 17
  },
  "regime": {
    "cycle_phase": "late_cycle",
    "risk_appetite": "neutral",
    "growth": "decelerating",
    "inflation": "sticky",
    "liquidity": "tightening",
    "fed_stance": "hawkish_hold",
    "regime_change_vs_prior": true
  },
  "consensus_matrix": [
    {
      "asset_class": "us_equities",
      "views": {
        "blackrock": "OW",
        "vanguard": "N",
        "pimco": "UW",
        "jpm": "OW",
        "goldman": "OW",
        "morgan_stanley": "UW",
        "bofa": "N",
        "marks": null,
        "gmo": "UW"
      },
      "consensus": "SPLIT",
      "agreement_score": 0.4,
      "conviction": "LOW"
    }
  ],
  "themes": [
    {
      "title": "AI capex cycle peaking",
      "firms_flagging": ["morgan_stanley", "gmo", "hussman"],
      "conviction": "MODERATE",
      "contrarian_view": "Goldman argues capex still in early innings"
    }
  ],
  "risks": [
    {
      "risk": "US fiscal crisis triggers bond vigilantes",
      "probability": "low",
      "impact": "severe",
      "flagged_by": ["gromen", "lyn_alden", "bridgewater"]
    }
  ],
  "portfolio_alignment": {
    "aligned_with_consensus": ["GLD", "SGOL"],
    "against_consensus": ["COPX", "IBIT"],
    "missing_consensus_overweights": ["intl_dm_equities"],
    "behavioral_flags": [
      {
        "position": "COPX",
        "flag": "averaging_down",
        "factor_id": 7,
        "note": "Stop breached, consensus UW on industrial metals"
      }
    ]
  },
  "action_implications": [
    {
      "position": "COPX",
      "current_pl_pct": -23.6,
      "consensus": "UW",
      "consensus_firms_agreeing": 6,
      "consensus_firms_total": 9,
      "behavioral_flags": ["averaging_down", "disposition_effect"],
      "implication": "reduce",
      "conviction": "HIGH",
      "counter_case": "Gromen/Lyn Alden argue commodity supercycle intact"
    }
  ]
}
```

---

## Workflow

```
1. Source  →  Fetch latest public research from Tier 1-3 sources via Firecrawl
2. Extract →  Pull key views, calls, data points per source
3. Synthesize → Cross-reference views, identify consensus/divergence
4. Anchor  →  Pull current positions from portfolio.db
5. Overlay →  Map consensus to positions + run behavioral flags from behavioral_report_v4.json
6. Recommend → Generate action implications where signals converge
7. Output  →  Write outlook-YYYY-MM.md + outlook-YYYY-MM.json
```

---

## File Structure

```
YaInvest/
├── outlooks/                          # New directory
│   ├── outlook-2026-03.md             # This month's strategy memo
│   ├── outlook-2026-03.json           # Structured companion data
│   └── archive/                       # Older than 6 months
├── outlook-sources.json               # Source registry with URLs + fetch config
├── outlook-template.md                # Section template with placeholders
└── symbol-asset-class-map.json        # Maps portfolio symbols to consensus matrix asset classes
```

---

## Source Registry Schema (outlook-sources.json)

```json
{
  "sources": [
    {
      "id": "blackrock",
      "name": "BlackRock Investment Institute",
      "tier": 1,
      "url": "https://www.blackrock.com/corporate/insights/blackrock-investment-institute",
      "fetch_method": "firecrawl",
      "expected_cadence": "weekly",
      "extract_hints": "Look for 'Weekly commentary' and 'Global weekly outlook' sections",
      "last_fetched": "2026-03-21",
      "last_fetched_content_date": "2026-03-17"
    },
    {
      "id": "fed_data",
      "name": "FRED Economic Data",
      "tier": 3,
      "url": "https://fred.stlouisfed.org",
      "fetch_method": "api",
      "series": ["DGS10", "DGS2", "CPIAUCSL", "UNRATE", "T10Y2Y"],
      "expected_cadence": "daily",
      "last_fetched": null,
      "last_fetched_content_date": null
    }
  ]
}
```

Fields: `id` (matches consensus matrix keys), `name`, `tier` (1/2/3), `url`, `fetch_method` (firecrawl / api / media_search), `expected_cadence`, `extract_hints` (guidance for extraction), `last_fetched`, `last_fetched_content_date`. Tier 3 data sources may include `series` for specific data series IDs.

### Consensus Matrix Conviction Derivation

The `conviction` field in each consensus matrix entry is derived from `agreement_score`:
- **HIGH:** agreement_score >= 0.7 (strong consensus)
- **MODERATE:** agreement_score 0.4–0.69 (leaning but not decisive)
- **LOW:** agreement_score < 0.4 (highly fragmented views)

---

## Symbol-to-Asset-Class Mapping (symbol-asset-class-map.json)

Maps portfolio symbols to consensus matrix asset classes. Maintained manually — update when new positions are opened.

```json
{
  "AAPL": "us_equities",
  "AMZN": "us_equities",
  "COPX": "commodities_industrial_metals",
  "GLD": "gold",
  "SGOL": "gold",
  "IBIT": "crypto",
  "HGT": "commodities_industrial_metals",
  "VT": "intl_dm",
  "SPY": "us_equities",
  "BTGD": "gold",
  "SQN": "us_equities"
}
```

The consensus matrix uses these asset class keys: `us_equities`, `intl_dm`, `em_equities`, `govt_bonds`, `credit`, `commodities_energy`, `commodities_industrial_metals`, `gold`, `crypto`, `cash`.

---

## Report Template (outlook-template.md)

The template is a markdown skeleton with section headers and placeholder markers. Generated once, reused each month:

```markdown
# Institutional Investment Outlook — {{MONTH}} {{YEAR}}

*Generated: {{DATE}} | Sources consulted: {{N}} of {{TOTAL}}*

---

## 1. Executive Summary
{{executive_summary}}

## 2. Macro Regime
{{macro_regime}}

## 3. Asset Class Views
{{asset_class_views}}

## 4. Consensus Matrix
{{consensus_matrix_table}}

## 5. Thematic Spotlight
{{thematic_spotlight}}

## 6. Risk Radar
{{risk_radar}}

## 7. Portfolio Alignment
{{portfolio_alignment}}

## 8. Behavioral Check
{{behavioral_check}}

## 9. Action Implications
{{action_implications}}

---
*Sources: {{source_list}}*
*Disclaimer: This is a personal research synthesis, not investment advice.*
```

Word counts per section are guidance (~200-500 words), not enforced limits. The synthesizer should aim for concision but not truncate meaningful analysis.

---

## Month-over-Month Diffing

The JSON companion enables structured diffing. Key tracked fields:

| Field | What a change means |
|-------|-------------------|
| `regime.cycle_phase` | Macro regime shift — most significant |
| `regime.risk_appetite` | Risk posture change |
| `regime.fed_stance` | Central bank pivot |
| `consensus_matrix[].consensus` | Firm consensus flipped on an asset class |
| `consensus_matrix[].views.*` | Individual firm changed its call |
| `risks[]` | New tail risks emerged or old ones dropped off |
| `portfolio_alignment.against_consensus[]` | Your positions moved in/out of alignment |

**Workflow:** When generating a new month's outlook, read the prior month's JSON (`outlook-YYYY-MM.json`) if it exists. Use it to:
1. Populate `regime.regime_change_vs_prior`
2. Detect SHIFTING consensus (firm view flips)
3. Generate the "what changed this month" bullets in the Executive Summary
4. Flag positions that moved from aligned to misaligned (or vice versa)

If no prior month exists (first run), set `regime_change_vs_prior: false` and skip diff-dependent content.

---

## Archive Policy

- **Trigger:** Manual. At the start of each session, check if any outlooks are >6 months old.
- **What moves:** Both `.md` and `.json` files move to `outlooks/archive/`.
- **Archived files remain accessible** for historical diffing — the archive directory is flat (no subdirectories).
- **No deletion.** Archived outlooks are never deleted.

---

## Portfolio Integration

**Data sources within the project:**
- `portfolio.db` → `position_snapshots` (latest) for current positions
- `portfolio.db` → `trades` for recent trade activity (last 30 days)
- `behavioral_report_v4.json` → 10-factor scores

**Alignment logic per position:**
1. Map position to asset class (e.g., COPX → commodities/industrial metals)
2. Look up consensus for that asset class in the matrix
3. Flag if overweight and consensus is underweight (or vice versa)
4. Check recent trades: added in last 30 days?
5. Cross-reference against behavioral factors:
   - Adding to a loser against consensus → Factor 7 (averaging down) + Factor 6 (chasing)
   - Holding a stop-breached position consensus is bearish on → Factor 1 (disposition)
   - Concentrated in an asset class consensus is UW → Factor 10 (correlation neglect)

**Action implication conviction scoring:**
- HIGH: consensus + risk rules + behavioral model all align
- MODERATE: two of three align
- LOW: only one signal, or mixed signals

**Boundaries:**
- The outlook is read-only intelligence. It does not modify portfolio.db or any existing files.
- Action implications are "institutional consensus implications" — the decision remains with the user.
- The behavioral overlay is a mirror, not a directive.
