# The RWA Distribution Gap on Mantle — Findings

**TL;DR: Mantle has solved RWA *supply* and not RWA *distribution*. 368 tokenized US equities are live on-chain; exactly 1 is genuinely distributed. 926 wallets hold any of them — 88% arrived in a single four-week window, and that window is now attributed: it was the Bybit ⇄ Mantle xStocks gateway opening (Apr 10, 2026). Those users are excellent demand — 98.3% still hold — but when the launch moment passed, acquisition collapsed to single digits per week. Mantle's RWA demand engine is one CEX gateway that only fires during moments. That is the gap — and it is fixable.**

*All numbers measured on-chain from raw `mantle.logs` Transfer events (coverage: Mantle genesis 2023-07-02 → 2026-07-02) in exact 256-bit integer arithmetic, cross-validated against the Routescan API (concentration matched to the decimal; holder counts within ~3%, live data being fresher). Snapshot: 2026-07-02. Every finding links to a public, re-runnable Dune query. Not financial advice.*

**Live dashboard:** https://dune.com/yeheskiel/mantle-rwa-distribution-tracker

---

## Findings

### F1 — 368 issued, 1 distributed
Backed's xStocks put **368 tokenized US equities** on Mantle (family launched 2025-12-01). Against falsifiable distribution thresholds (holders > 500, top-1 wallet < 75%):
- **1 token passes: NVDAx** — 612 holders / top-1 74.55% on the Dune snapshot; 629 holders on the same-day live Routescan check (both sources agree the thresholds are crossed)
- **15 of 368 (4.1%)** have even 10 holders
- The median xStock has ~1–3 holders — *issued, not adopted*

→ [League query 7863671](https://dune.com/queries/7863671) · [Summary 7863679](https://dune.com/queries/7863679) · live cross-check: [agent digest 2026-07-02](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker/blob/main/agent/reports/2026-07-02.md)

### F2 — Adoption is event-driven — and the event is now identified: the Bybit gateway
Weekly first-time-holder counts show three regimes:
- **Dec 2025 – mid-Mar 2026:** ~nothing. 3–6 total holders for 3.5 months after launch.
- **Apr 13 – May 10, 2026:** an explosion — **+816 wallets in 4 weeks** (353 in the single week of Apr 13). **88% of all holders ever were acquired in this one window.**
- **Since May 18:** +1 to +11 new holders per week. Even the SpaceX token launch (Jun 11) added only ~50.

**Attribution (three independent lines of evidence):**
1. **On-chain fingerprint:** the cohort's entry transfers are overwhelmingly *direct `token.transfer()` calls initiated by a third party* (`tx_to` = the token contract, zero self-initiated) — the CEX-withdrawal pattern, not DEX buys and not an airdrop (amounts vary widely, stddev ≈ 14).
2. **The distributor:** one hot wallet, `0x5888…836c`, delivered first xStocks to **462 of the 816 cohort wallets**, across 7 tickers, active **every single day of Apr 13 – May 10** — then stopped. The wallet has 2.76M lifetime transactions on Mantle and appears in **Bybit's Proof-of-Reserves reports** for the Mantle network: it is Bybit's Mantle hot wallet.
3. **The off-chain event:** on **Apr 10, 2026** Mantle, Bybit and Backed announced the xStocks integration — Bybit enabled xStocks **deposits and withdrawals on the Mantle network** ([Bybit announcement](https://announcements.bybit.com/en/article/bybit-now-supports-xstocks-deposits-and-withdrawals-on-mantle-bltd6af69aacd874633/), [The Block](https://www.theblock.co/post/378030/bybit-backed-xstocks-tokenized-nvidia-mstr-mantle)). The on-chain wave starts the first Monday after.

The spike proves demand *can* be activated on Mantle — via the CEX gateway. The collapse proves the gateway only fires during launch moments; there is **no always-on funnel** behind it.

→ [Trend 7865851](https://dune.com/queries/7865851) · [Mechanism 7866292](https://dune.com/queries/7866292) · [Distributors 7866312](https://dune.com/queries/7866312)

### F2b — The Bybit-gateway users are excellent demand: 98.3% still hold
Of the 816 wallets that got their first xStock in the Bybit window, **802 (98.3%) still hold today** — buy-and-hold retail, not incentive farmers. Contrast: the June SPCXx launch came with **Merchant Moe's "Project X" (up to 100,000 MNT in LP rewards)** — it produced pool liquidity and trading churn (F5) but **~zero new holders**. Liquidity incentives buy liquidity; the CEX gateway buys *holders*. Mantle currently funds the former and leaves the latter to launch-day marketing.

→ [Retention 7866325](https://dune.com/queries/7866325) · [Project X press release](https://chainwire.org/2026/06/12/mantle-and-xstocks-bring-tokenized-spacex-spcxx-to-fluxion-merchant-moe-as-historys-largest-ipo-goes-live/)

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

→ [Agent digest 2026-07-02 (committed by CI)](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker/blob/main/agent/reports/2026-07-02.md) · [CI run history](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker/actions)

---

## Recommendations for Mantle — each with the why and the evidence

### R1 — Make the CEX gateway always-on, not launch-day
**Why:** the only funnel that has ever produced RWA holders at scale on Mantle is CEX → on-chain withdrawal, and it only ran while the launch moment lasted. Each active gateway week delivered ≈ +200–350 holders; when it cooled, acquisition fell to single digits. Recurring, localized gateway campaigns — Indonesia's ~26M retail stock investors are the natural wedge, and Bybit already operates there — would compound what is currently a one-off.
**Evidence:** 88% of all holders arrived Apr 13 – May 10 ([trend 7865851](https://dune.com/queries/7865851)); the deliveries came from Bybit's Mantle hot wallet ([distributors 7866312](https://dune.com/queries/7866312), [Bybit PoR](https://www.bybit.com/common-static/cht-static/por/Bybit_PoR_Audit_2026_Feb_26.pdf)); the window opens right after the [Apr 10 Bybit×Mantle announcement](https://announcements.bybit.com/en/article/bybit-now-supports-xstocks-deposits-and-withdrawals-on-mantle-bltd6af69aacd874633/); post-window growth is +1–11/week ([trend 7865851](https://dune.com/queries/7865851)).

### R2 — Rebalance incentives from liquidity to first-holder acquisition
**Why:** the two experiments the ecosystem has already run point the same way. The CEX-gateway users cost no incentives and 98.3% of them still hold; the LP-rewards campaign (Project X, up to 100k MNT) produced pool depth and trading churn but ~zero new holders. Budget follows what it measures — right now it buys liquidity, not distribution.
**Evidence:** retention 802/816 = 98.3% ([retention 7866325](https://dune.com/queries/7866325)); SPCXx post-launch = thousands of transfers but ≤19 active receivers/day and 26 holders ([activity 7863657](https://dune.com/queries/7863657), [concentration 7863618](https://dune.com/queries/7863618)); the June SPCXx launch shipped with [Project X LP rewards](https://chainwire.org/2026/06/12/mantle-and-xstocks-bring-tokenized-spacex-spcxx-to-fluxion-merchant-moe-as-historys-largest-ipo-goes-live/) yet the holder trend barely moved that month ([trend 7865851](https://dune.com/queries/7865851)).

### R3 — Fix the measurement stack (Fluxion indexing + xStock price feeds)
**Why:** builders and BD cannot optimize what they cannot see. Mantle's flagship RWA venue is invisible to the standard analytics stack, and tokenized-equity liquidity cannot be quoted in USD at all — this study had to fall back to trade counts.
**Evidence:** `dex.trades` on Mantle covers merchant_moe / agni / uniswap / fusionx only — **no Fluxion** — and `amount_usd` is NULL for xStock trades ([DEX 7863658](https://dune.com/queries/7863658); methodology notes in the [repo README](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker#methodology--honesty-notes)).

### R4 — Adopt *external float %* and *holders* as the RWA KPIs, not TVL/supply
**Why:** supply is an issuer decision — SPCXx's entire supply history is one 150k mint and one 120k burn — so supply-side KPIs measure the issuer, not the market. Distribution metrics are falsifiable, cheap to monitor continuously, and they already caught the one genuine success.
**Evidence:** SPCXx mint/burn history ([7863661](https://dune.com/queries/7863661)) vs external float stuck at 0.8% ([7863645](https://dune.com/queries/7863645)); NVDAx crossing both thresholds was detected automatically by the tracker's first multi-token run ([agent digest + alerts, 2026-07-02](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker/blob/main/agent/reports/2026-07-02.md)) — and the crossing dates to the Bybit window ([trend 7865851](https://dune.com/queries/7865851)), the causal chain working end-to-end once.

## Reuse this research

Everything is public and parameterized:
- **Dashboard:** https://dune.com/yeheskiel/mantle-rwa-distribution-tracker — token selector switches every single-token panel; fork any query and change `token_address` to track any ERC-20 on Mantle.
- **Queries:** `dune/q1…q13.sql` in this repo = the exact deployed SQL, with live query IDs in each header (q11–q13 = the spike attribution forensics: entry mechanism, distributor identity, cohort retention).
- **Agent:** `agent/` — stdlib-only Python (no pip). Weekly digest + threshold alerts + Dune/live cross-check. Works for any token on any Routescan-supported chain by editing `config.json` (`chain_id`, `tokens`). GitHub Actions cron included and verified.
- **Method:** exact `decimal(38,0)` balance reconstruction (no float error, no dust hack), name-based token-family discovery, `creation_traces` contract flagging, first-activity-vs-first-receipt acquisition bucketing.

## Method notes & caveats

- Balances reconstructed from ERC-20 Transfer logs only; exact integer math; zero address and issuer wallet (`0x5F7A…a2aD`, verified top holder across the family) excluded from holder counts.
- League-table external float uses each token's top holder as the issuer proxy.
- Acquisition "first activity" = first tx a wallet *initiated* that emitted a log; pure MNT transfers without logs are invisible (undercounts wallet age slightly).
- Trend (926) vs acquisition (923) totals differ by 3 wallets (0.3%) — anomalous per-token flows on non-standard tokens; immaterial.
- Routescan live counts run ~0–3% above the same-day Dune snapshot (fresher data); the agent's pagination is exact up to 1,000 holders, then reports a lower bound (`N+`).
- Spike attribution rests on three independent lines (transfer fingerprint, Bybit PoR wallet match, dated announcement); the long tail of smaller senders (~40% of cohort entries) is unattributed — plausibly other exchange wallets, OTC desks, or re-distribution, and several show identical payout sizes (e.g. 10.0772 tokens from ≥4 different wallets), suggesting one programmatic upstream source.
- Retention is measured as "still holds any xStock balance > 0", not "still active on Mantle"; a holder can be retained and dormant.

---

## Evidence index — every number, one click away

**Dashboard (all panels):** https://dune.com/yeheskiel/mantle-rwa-distribution-tracker

| # | Dune query (public, re-runnable) | Supports |
|---|---|---|
| [7863679](https://dune.com/queries/7863679) | Ecosystem summary counters (368 / 15 / 1) | F1 |
| [7863671](https://dune.com/queries/7863671) | xStock league — every token ranked, verdicts | F1 |
| [7865851](https://dune.com/queries/7865851) | Holders over time — the 4-week window, 926 total | F2, R1, R2, R4 |
| [7865842](https://dune.com/queries/7865842) | Holder acquisition — 45.6% arrived with first xStock, median 459-day OGs | F3 |
| [7866292](https://dune.com/queries/7866292) | Spike entry mechanism — CEX-withdrawal fingerprint | F2, R1 |
| [7866312](https://dune.com/queries/7866312) | Spike distributors — Bybit hot wallet `0x5888…836c`, 462/816 wallets | F2, R1 |
| [7866325](https://dune.com/queries/7866325) | Bybit-window cohort retention — 98.3% | F2b, R2 |
| [7863618](https://dune.com/queries/7863618) | Concentration (parameterized; SPCXx default 99.2% top-1) | F4, R2 |
| [7863645](https://dune.com/queries/7863645) | External float over time (SPCXx 0 → 0.8%) | F4, R4 |
| [7863651](https://dune.com/queries/7863651) | Holders over time, single token | F4 |
| [7863661](https://dune.com/queries/7863661) | Mint / burn / net supply (one mint, one burn) | F4, R4 |
| [7863657](https://dune.com/queries/7863657) | Daily activity — transfers vs distinct wallets | F5, R2 |
| [7863658](https://dune.com/queries/7863658) | DEX activity — Merchant Moe only; `amount_usd` NULL | F6, R3 |

**External sources**
- [Bybit announcement — xStocks deposits & withdrawals on Mantle (Apr 10, 2026)](https://announcements.bybit.com/en/article/bybit-now-supports-xstocks-deposits-and-withdrawals-on-mantle-bltd6af69aacd874633/)
- [The Block — Bybit and Backed partner to bring tokenized stocks to Mantle](https://www.theblock.co/post/378030/bybit-backed-xstocks-tokenized-nvidia-mstr-mantle)
- [Mantle × Bybit × Backed joint press release (PR Newswire)](https://www.prnewswire.com/news-releases/mantle-becomes-one-of-the-first-ethereum-l2s-to-bring-tokenized-equities-to-on-chain-liquidity-with-xstocks-and-bybit-302739354.html)
- [Bybit Proof-of-Reserves audit (Mantle wallets, incl. `0x5888…836c`)](https://www.bybit.com/common-static/cht-static/por/Bybit_PoR_Audit_2026_Feb_26.pdf)
- [Chainwire — SPCXx live on Fluxion & Merchant Moe; Project X, up to 100k MNT LP rewards (Jun 12, 2026)](https://chainwire.org/2026/06/12/mantle-and-xstocks-bring-tokenized-spacex-spcxx-to-fluxion-merchant-moe-as-historys-largest-ipo-goes-live/)
- Routescan v2 API (live holder pagination & cross-checks): `api.routescan.io/v2/network/mainnet/evm/5000/…`
- [Agent digests, committed by CI](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker/tree/main/agent/reports) · [CI runs](https://github.com/yeheskieltame/mantle-rwa-distribution-tracker/actions)

*Author: Yeheskiel Yunus Tame ([@YeheskielTame](https://x.com/YeheskielTame)) — UKDW Blockchain Club, OwnaFarm. Part of the Mantle Research Challenge. Not financial advice.*
