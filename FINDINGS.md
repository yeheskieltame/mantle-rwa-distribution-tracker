# The RWA Distribution Gap on Mantle — Findings

**TL;DR: Mantle has solved RWA *supply* and not RWA *distribution*. 368 tokenized US equities are live on-chain; exactly 1 is genuinely distributed. 926 wallets hold any of them — and 88% of those wallets arrived in a single four-week window. When the moment passed, acquisition collapsed to single digits per week. The demand side is missing — and that is fixable.**

*All numbers measured on-chain from raw `mantle.logs` Transfer events (coverage: Mantle genesis 2023-07-02 → 2026-07-02) in exact 256-bit integer arithmetic, cross-validated against the Routescan API (matched to the decimal). Snapshot: 2026-07-02. Every finding links to a public, re-runnable Dune query. Not financial advice.*

**Live dashboard:** https://dune.com/yeheskiel/mantle-rwa-distribution-tracker

---

## Findings

### F1 — 368 issued, 1 distributed
Backed's xStocks put **368 tokenized US equities** on Mantle (family launched 2025-12-01). Against falsifiable distribution thresholds (holders > 500, top-1 wallet < 75%):
- **1 token passes: NVDAx** (629 holders, top-1 74.55%)
- **15 of 368 (4.1%)** have even 10 holders
- The median xStock has ~1–3 holders — *issued, not adopted*

→ [League query 7863671](https://dune.com/queries/7863671) · [Summary 7863679](https://dune.com/queries/7863679)

### F2 — Adoption is event-driven, not organic
Weekly first-time-holder counts show three regimes:
- **Dec 2025 – mid-Mar 2026:** ~nothing. 3–6 total holders for 3.5 months after launch.
- **Apr 13 – May 10, 2026:** an explosion — **+816 wallets in 4 weeks** (353 in the single week of Apr 13). **88% of all holders ever were acquired in this one window.**
- **Since May 18:** +1 to +11 new holders per week. Even the SpaceX token launch (Jun 11) added only ~50.

The spike proves demand *can* be activated on Mantle. The collapse proves there is **no always-on distribution funnel** — only moments.

→ [Trend query 7865851](https://dune.com/queries/7865851)

### F3 — RWAs do acquire new users — but the funnel is tiny
Classifying all 923 current external holder wallets by their on-chain history:

| Wallet origin | Wallets | Share |
|---|---|---|
| Existing Mantle user (median **459 days** on Mantle before first xStock) | 458 | 49.6% |
| **Arrived on Mantle with their first xStock** | 421 | 45.6% |
| Contract (DEX pool / vault) | 30 | 3.3% |
| Passive receiver (never initiated a tx) | 14 | 1.5% |

Two implications: (a) tokenized stocks are a real user-acquisition product — nearly half the holders are wallets that showed up *for the RWA*; (b) **421 new wallets in 7 months** is the entire acquisition result. For scale: Indonesia alone has ~26M retail stock investors.

→ [Acquisition query 7865842](https://dune.com/queries/7865842)

### F4 — Case study SPCXx: supply without demand
The SpaceX xStock (launched Jun 11, 2026, heavily promoted): net supply 30,000 tokens, **99.2% still in the issuer wallet**. External float after 3 weeks: **0.8%**. 26 holders; 7 hold ≥ 1 whole token. Supply was set by exactly one mint (150k) and one burn (120k) — issuer operations, not demand.

→ [Concentration 7863618](https://dune.com/queries/7863618) · [Float over time 7863645](https://dune.com/queries/7863645) · [Mint/burn 7863661](https://dune.com/queries/7863661)

### F5 — Activity ≠ adoption
SPCXx shows 5,686 transfers in 21 days (spikes of 1,400+/day) — but never more than ~19 distinct receiving addresses in a day. High transfer counts are churn (arb bots, issuer ops), not users. Any RWA metric based on transaction counts will overstate adoption.

→ [Daily activity 7863657](https://dune.com/queries/7863657)

### F6 — The ecosystem cannot even measure RWA liquidity yet
- SPCXx trades on **one venue** (Merchant Moe; 418 trades, peaked at 208/week, decaying to ~42).
- **Fluxion — Mantle's RWA-focused DEX — is not indexed in Dune's `dex.trades`** spellbook at all.
- **xStocks are missing from the DEX price oracle**, so `amount_usd` is NULL — USD liquidity of tokenized equities on Mantle is *unmeasurable* with standard tooling.

The measurement infrastructure gap is itself an adoption blocker: builders can't optimize what they can't see.

→ [DEX query 7863658](https://dune.com/queries/7863658)

### F7 — First threshold crossing, detected by the tracker
On its first multi-token run (2026-07-02 11:40 UTC), the automation agent alerted that **NVDAx crossed both falsifiable thresholds** (top-1 74.553% < 75%; holders 629 > 500) — the first xStock on Mantle to become *distributed* by pre-registered criteria. The framework works: thresholds set in advance, crossing detected automatically, receipts on-chain.

---

## What Mantle can do with this

1. **Build an always-on funnel, not campaigns.** F2 shows a 4-week moment created 88% of all RWA holders, then acquisition died. The demand side needs permanent rails — localized on-ramps, education, and retail UX in target markets (Indonesia's 26M retail investors are the natural wedge) — not one-off incentive spikes.
2. **Fix the measurement stack.** Get Fluxion into the Dune spellbook and xStocks into price feeds (F6). Today nobody — including Mantle — can quote USD liquidity for tokenized equities on Mantle.
3. **Adopt *external float %* and *holders* as the RWA KPIs, not TVL/supply.** Supply is an issuer decision (F4); distribution is the market's verdict. The thresholds here (>500 holders, top-1 <75%) are falsifiable and now continuously monitored.
4. **Study NVDAx.** One token of 368 crossed the line (F1, F7). Understanding *why* NVDAx distributed — venue, incentives, timing within the April window — is the cheapest playbook Mantle can buy.

## Reuse this research

Everything is public and parameterized:
- **Dashboard:** https://dune.com/yeheskiel/mantle-rwa-distribution-tracker — token selector switches every single-token panel; fork any query and change `token_address` to track any ERC-20 on Mantle.
- **Queries:** `dune/q1…q10.sql` in this repo = the exact deployed SQL, with live query IDs in each header.
- **Agent:** `agent/` — stdlib-only Python (no pip). Weekly digest + threshold alerts + Dune/live cross-check. Works for any token on any Routescan-supported chain by editing `config.json` (`chain_id`, `tokens`). GitHub Actions cron included and verified.
- **Method:** exact `decimal(38,0)` balance reconstruction (no float error, no dust hack), name-based token-family discovery, `creation_traces` contract flagging, first-activity-vs-first-receipt acquisition bucketing.

## Method notes & caveats

- Balances reconstructed from ERC-20 Transfer logs only; exact integer math; zero address and issuer wallet (`0x5F7A…a2aD`, verified top holder across the family) excluded from holder counts.
- League-table external float uses each token's top holder as the issuer proxy.
- Acquisition "first activity" = first tx a wallet *initiated* that emitted a log; pure MNT transfers without logs are invisible (undercounts wallet age slightly).
- Trend (926) vs acquisition (923) totals differ by 3 wallets (0.3%) — anomalous per-token flows on non-standard tokens; immaterial.
- Routescan live counts run ~0–3% above the same-day Dune snapshot (fresher data); the agent's pagination is exact up to 1,000 holders, then reports a lower bound (`N+`).
- The Apr 13 – May 10 spike is measured, not explained: attribution (campaign, listing, incentive) is an open question worth answering before designing the next push.

---

*Author: Yeheskiel Yunus Tame ([@YeheskielTame](https://x.com/YeheskielTame)) — UKDW Blockchain Club, OwnaFarm. Part of the Mantle Research Challenge. Not financial advice.*
