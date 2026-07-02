# Mantle RWA Distribution Tracker

Dune dashboard + automation agent that measure **distribution, not just supply**, for tokenized real-world assets (RWA) on Mantle. Follow-up to the Research Challenge thesis: **supply ≠ adoption.**

### ▶ Live dashboard: https://dune.com/yeheskiel/mantle-rwa-distribution-tracker
### ▶ Research findings: [FINDINGS.md](FINDINGS.md)

**Headline (Mantle, snapshot 2026-07-02):**
- **368** xStocks issued on Mantle — only **1** (NVDAx) clears the 500-holder distribution bar; **15** have ≥10 holders.
- **926** wallets hold any xStock — **88% acquired in a single 4-week spike**, now attributed on-chain to the **Bybit ⇄ Mantle xStocks gateway opening (Apr 10, 2026)**: Bybit's Mantle hot wallet delivered first xStocks to 462 of the 816 cohort wallets, then the wave stopped.
- Those users stuck: **98.3% of the Bybit-window cohort still holds** — CEX-gateway demand is buy-and-hold, not farming. Meanwhile LP incentives (Project X, 100k MNT) produced churn, not holders.
- **45.6%** of holder wallets arrived on Mantle *with* their first xStock (421 new wallets in 7 months); the rest are OG Mantle users (median **459 days** old).
- **SPCXx** (SpaceX xStock): 30,000 net supply, **99.2% in the issuer wallet → external float 0.8%**, 26 holders.

*The supply is here. The demand — 26 million Indonesian retail investors among them — is not. That gap is the opportunity.*

---

## What it shows

The **hero metric is external float %** — the share of net supply held *outside* the issuer wallet. High supply + tiny external float = *issued, not adopted*.

**Falsifiable thresholds** — a token is genuinely distributed when: holders **> 500** · top-1 wallet **< 75%** · DEX liquidity **> $100K** (where priced).

Dashboard sections & queries (all raw `mantle.logs`, exact 256-bit math):

Dashboard sections: **① the adoption gap** · **② attribution — who opened the tap** · **③ single-token deep-dive** (token selector) · **④ issuer-driven supply** · **⑤ activity & liquidity**. Every finding in [FINDINGS.md](FINDINGS.md) has a live panel:

| § | Query | Live |
|---|-------|------|
| ① | Ecosystem summary — issued vs adopted counters | [7863679](https://dune.com/queries/7863679) |
| ① | xStock ecosystem league (every xStock, ranked) | [7863671](https://dune.com/queries/7863671) |
| ① | xStock holders over time — the 4-week spike | [7865851](https://dune.com/queries/7865851) |
| ① | Holder acquisition — new wallets vs recycled users | [7865842](https://dune.com/queries/7865842) |
| ② | Spike mechanism — how the cohort got its first xStock | [7866292](https://dune.com/queries/7866292) |
| ② | Spike distributors — the Bybit hot wallet identified | [7866312](https://dune.com/queries/7866312) |
| ② | Bybit-window cohort retention (98.3%) | [7866325](https://dune.com/queries/7866325) |
| ③ | Concentration (holders, top-1 %, external float, net supply) | [7863618](https://dune.com/queries/7863618) |
| ③ | External float over time *(hero)* | [7863645](https://dune.com/queries/7863645) |
| ③ | Holders over time | [7863651](https://dune.com/queries/7863651) |
| ④ | Mint / burn / net supply over time | [7863661](https://dune.com/queries/7863661) |
| ⑤ | Daily activity | [7863657](https://dune.com/queries/7863657) |
| ⑤ | DEX activity (Merchant Moe) | [7863658](https://dune.com/queries/7863658) |

Queries are parameterized (`token_address`, `issuer_wallet`, default SPCXx) — fork any of them and change the parameter to track a different RWA.

## Methodology & honesty notes

- **Exact 256-bit integer math** (`decimal(38,0)`, not floating point). `double` loses integer precision above ~2^53, which forces a fragile `>1e6` "dust" hack; with exact math a zero balance is exactly zero and **holder counts are exact**. Cross-checked against the Routescan API — top-1 99.2% / 26 holders matched to the decimal.
- **Ecosystem universe** = `tokens.erc20` where `name LIKE '%xStock%'` (not `symbol LIKE '%x'`, which catches junk like `100x`, `55X`, `Arcane Flux`).
- **External float in the league** uses each token's top holder as the issuer/treasury proxy (no per-token issuer map needed).
- **DEX**: `dex.trades` covers Merchant Moe, Agni, Uniswap, FusionX on Mantle. **Fluxion is not indexed yet.** SPCXx is **not in the DEX price oracle**, so its DEX panel shows **trade counts**, not USD.
- Every number carries its source + timestamp. **Not financial advice.**

## Automation agent (`agent/`)

Stdlib-only Python (no `pip` — students can run it as-is). Tracks **7 xStocks** out of the box (NVDAx, TSLAx, CRCLx, GOOGLx, COINx, SPCXx, AAPLx). One run does three jobs:

- **DIGEST** — weekly markdown brief to `agent/reports/YYYY-MM-DD.md` (raw material for a weekly X thread). Holder counts are exact up to 1,000 (paginated Routescan; `N+` = lower bound beyond that).
- **ALERTS** — webhook ping when a falsifiable threshold is crossed: top wallet < 75% 🚨, holders reach 500+ 🎯, top-1 drops ≥1 pt 📉, holders jump ≥25 📈. *(On its first multi-token run the agent detected NVDAx crossing both thresholds — see [FINDINGS.md](FINDINGS.md) F7.)*
- **SYNC** — cross-checks the live Routescan number against the Dune concentration query and flags divergence > 2 pts.

The alert path is end-to-end tested: a local stdlib webhook receiver + seeded state force every alert type through a real HTTP POST.

Run locally: `python3 agent/tracker_agent.py`

Automated (free) via GitHub Actions — the workflow is already included at `.github/workflows/tracker.yml`: full digest every Monday 08:00 WIB, daily alert check the rest of the week, plus a manual "Run workflow" button. Optional repo secrets: `ALERT_WEBHOOK_URL` (Discord/Slack), `DUNE_API_KEY` (enables the sync-check; without it the agent silently uses Routescan only).

## Add a new token

Edit `agent/config.json` → add an entry under `tokens` (address + issuer wallet from Mantlescan). On Dune, just change the query `token_address` parameter.

*Not financial advice.*
