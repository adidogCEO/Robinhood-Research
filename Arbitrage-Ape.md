# Arbitrage Ape ($AA) — Due Diligence Report

**Generated:** 2026-09-07 ~19:55 UTC (v4 refresh)  
**Chain:** Robinhood Chain (`4663`) — RPC-verified  
**Exact token:** `0xe2dae1c072b9f66bed999873b94798b2152af7e1`  
**Scope:** Product mechanics, custody/permission map, risks, onchain-verified payouts, skill-grade surface scoring  
**Format:** Single self-contained Markdown (status + full payout history + target pin embedded)  
**Skill:** [evm-token-due-diligence](sand-workflow:evm-token-due-diligence)  
**Prior:** v3 merged 2026-09-05 (pin `55498712`)

### State pin
| Field | Value |
|---|---|
| Label | `state_head` |
| Block | `57099428` |
| Block hash | `0x80e6ce7ba61459ef97604844b2c75a7123a183531275b0bb53fa1d5b2366f15c` |
| UTC | `2026-09-07T19:54:31Z` |

---

## Executive summary

Arbitrage Ape remains a **live, programmatic** market-making / dislocation desk on Robinhood Chain. Since the 2026-09-05 report, holder **USDG distributions roughly doubled** (~$48.8k → ~$103.5k) while the desk stayed in `live` mode on a ~15-minute cadence. Vault cash on-chain is higher; **LP principal PnL is more negative**. Custody risk is unchanged.

| Question | Answer |
|---|---|
| Real onchain capital + automated execution? | **Yes** |
| Holder payments real USDG? | **Yes** (RPC-verified latest round + live API) |
| Empty “waiting for keeper” UI = inactive? | **No** — prefer `/api/*` |
| Trustless / rug-resistant vault? | **NO-GO** — `withdraw` still present; no timelock |

### Delta vs 2026-09-05 (v3)

| Metric | v3 | v4 now |
|---|---:|---:|
| Distributed USD | ~$48.8k | **~$103,531** |
| Realized profit | ~$49.1k | **~$118,748** |
| Vault USDG (API / on-chain) | ~$18.1k | **~$60,742 / $60,073** |
| LP principal PnL | ~$-37.0k | **~$-98,817** |
| Dist rounds / unique txs | 164 / 561 | **253 / 978** |
| $AA mark / mcap | ~$0.00179 / ~$1.79M | **~$0.00356 / ~$3,495,769** |
| Vault $AA share | ~10.7% | **~12.03%** (on-chain) |

### Surface scores (pinned state)

| Surface | Rating |
|---|---|
| Token controls | Moderate concern (Pons v2 token verified; vault is control plane) |
| Canonical LP-principal custody | High concern (vault/`exec` funded LP; owner can withdraw) |
| Side-pool removal risk | Unknown / elevated |
| Sellability and exit depth | Unknown (coverage — no holder-sized quotes this run) |
| Current concentration | Elevated (vault ~12.0% `$AA`) |
| Historical launch integrity | Partial (Pons v2 confirmed; SIZE-cohort not completed) |
| Admin / treasury / reward custody | **Critical** |
| Reward accounting and liveness | Strong (operational; still paying) |
| Utility and redemption rights | Weak for holders |
| External dependencies | High |
| Development and disclosure | Mixed (APIs good; vault unverified) |

**Do not average** the critical owner-withdraw finding with “payments are real.”

---

## 1. Product overview

| Item | Detail |
|---|---|
| Site | https://www.arbitrageape.app/ |
| Method docs | https://www.arbitrageape.app/docs |
| X | https://x.com/ArbitrageApe |
| Vault / fund | `0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98` |
| `$AA` fee token | `0xe2dae1c072b9f66bed999873b94798b2152af7e1` — **Arbitrage Ape** / `AA`, 18 decimals, 1B supply; Blockscout name `PonsV2LauncherToken` (verified) |
| USDG | `0x5fc5360d0400a0fd4f2af552add042d716f1d168` — **6 decimals** |
| Keeper (EOA) | `0xdef933cfaeb2a2af516df1eaa2101b00b7f77af6` |
| Owner | `0x12b46b7746af4a902fd55198f09ecfd8c4d42956` (`owner()` eth_call at pin) |
| Explorers | [Blockscout](https://robinhoodchain.blockscout.com/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98) · [Robinscan](https://robinscan.io/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98) |
| Live APIs | `/api/status`, `/api/distributions`, `/api/trades`, `/api/positions`, … |

**Live snapshot (API at report generation; pin `57099428`):**

| Metric | Value |
|---|---|
| Mode | `live` |
| Vault ETH | ~1.592740 ETH (on-chain ~1.592740) |
| Vault USDG | ~**$60,742.09** API · **$60,072.74** on-chain |
| Vault `$AA` (onchain) | ~**120,307,567** (~12.03% supply) |
| Realized profit | ~$118,747.92 |
| Distributed (API) | ~**$103,530.98** |
| Pending owed | ~$846.92 |
| Fee flow claimed (lifetime) | ~**$227,563.31** |
| LP principal PnL (API) | **~$-98,817.35** |
| Operator PnL (API) | ~$-3,283.014486199645 |
| Instruments / eligible | ~194 / ~92 |
| Pools tracked v3/v4 | 387 / 7836 |
| `$AA` mark (API) | ~$0.003555004227908025 · mcap ~$3,495,769 |

## 2. How it works

Unchanged from v3 in substance: Pons v2 creator-fee → vault inventory → sell dislocations via keeper `exec` → accrue realized pot → ~15m `distribute` USDG to eligible `$AA` holders. Prefer live `/api/*` over empty SSR chrome.

### 2.4–2.5 Onchain paths (still observed)
- Keeper → vault **`exec(address,uint256,bytes)`** (`0x0565bb67`)
- Keeper → vault **`distribute(address,address[],uint256[])`** (`0x15270ace`)

## 3. Custody and permission map (bytecode-level)

Vault **source still unverified**. At pin `57099428` runtime still contains:

| Function | Selector | Present |
|---|---|---|
| `withdraw(address,address,uint256)` | `0xd9caed12` | **Yes** |
| `exec(address,uint256,bytes)` | `0x0565bb67` | **Yes** |
| `distribute(...)` | `0x15270ace` | **Yes** |

| Role | Address | Notes |
|---|---|---|
| `owner()` | `0x12b46b7746af4a902fd55198f09ecfd8c4d42956` | unchanged; `pendingOwner()` empty |
| `keeper()` | `0xdef933cfaeb2a2af516df1eaa2101b00b7f77af6` | matches distribute tx `from` |

**Important:** Daily caps (if any) constrain the **keeper distribution path**. They do **not** neutralize **`owner.withdraw`**.

## 4. Risks

### Critical
1. **Instant owner drain** — `withdraw` with no timelock (still true at v4 pin).
2. **Unverified custody** — no public verified vault source; no audit found.
3. **Anonymous / thin social accountability**.

### High
4. **Single keeper** — stop / compromise / malicious `setKeeper`.
5. **Economic fragility** — LP principal PnL worsened (~$-37k → ~$-99k) while distributions continued.
6. **Circular / reflexive revenue** risk on fee-token MM and creator-fee loop.

### Product / market / UX
7–10. Same as v3 (thin pool edge, oracle, `$AA` ≠ NAV, empty SSR vs live APIs).

---

## 5. Payout rounds — desk-wide totals (refreshed)

| Metric | Value |
|---|---|
| Distribution rounds | **253** |
| Onchain payout transaction hashes | **978** unique |
| Rounds using multiple txs | **203** |
| Sum of API round totals | **$103,355.87** |
| Desk `distributedUsd` | **$103,530.98** |
| First round | 2026-09-04 13:19 UTC · $0.8666 · 33 holders |
| Latest round | 2026-09-07 19:37 UTC · $477.820844 · 851 holders · 5 txs |
| Largest round | 2026-09-07 02:08 UTC · **$3,929.99** · 1316 holders |
| Cadence | ~15–18 minutes (still live) |

### Size distribution (by round `totalUsd`)

| Band | Rounds | Sum USD |
|---|---:|---:|
| < $1 | 11 | $8.22 |
| $1–10 | 24 | $70.27 |
| $10–100 | 39 | $2,106.71 |
| $100–500 | 110 | $36,614.72 |
| $500–1,000 | 50 | $33,432.74 |
| ≥ $1,000 | 19 | $31,123.22 |

Full feed: https://www.arbitrageape.app/api/distributions

---

## 6. Onchain-verified payout samples

### A–C) Historical samples from v3
Earliest / large multi-tx samples from 2026-09-04–05 remain valid historical evidence (see prior report / Appendix D hashes).

### D) Latest round — full multi-tx (v4 pin) — exact match

| Field | Value |
|---|---|
| Time | 2026-09-07 19:37 UTC |
| API total | **$477.820844** |
| Holders | 851 |
| Tx count | 5 |
| Onchain USDG from vault | **$477.820844** |
| Δ API vs onchain | ~0 |
| Caller | keeper → vault · selector **`distribute` (`0x15270ace`)** · all success |

| Tx | USDG from vault | Recipients |
|---|---:|---:|
| `0x11ff524b78f7695fe7f9297f3d31890d7918a6ee1ec122bb4192a9498a316e81` | **$429.224566** | 200 |
| `0x72548676e8874013e618b029d7da89f69b515bee17b40f2d5ab119730427b04c` | **$33.387370** | 200 |
| `0x8dd7ddf699bd31d4d811fa873075298b3732634aef17154cdd13837fbae8ba38` | **$10.627633** | 200 |
| `0xfdee3fac605d58c73a4a0de26d8a421565891a613c46d0dac2cb7c1f15e1693e` | **$4.014524** | 200 |
| `0x99c4de8c08dd401e260e7407a53c4d54b6bd9a48e08da1cbe79b2d507b008acf` | **$0.566751** | 51 |

---

## 7. Holder distribution mechanics

Unchanged: pro-rata eligible `$AA`, exclude AMM/desk, multi-tx batching ~200 recipients/tx on large rounds.

---

## 8. Concentration (on-chain vault pin)

| Address | ~% supply | Note |
|---|---:|---|
| `0x0feB08fcF34F5c0e270261bC79BaFC0F8eb87f98` | **12.03%** | vault (pin balance) |

Full top-holder page refresh was not re-scraped this run (explorer Cloudflare coverage limit). Prior v3 page had vault ~10.7% and Uniswap v4 PoolManager as pool inventory.

---

## 9. Finding-to-evidence ledger (v4)

| ID | Proposition | Evidence | Confidence | Stale if |
|---|---|---|---|---|
| F1 | Chain is `4663` | `eth_chainId` at pin | High | other chain |
| F2 | Exact `$AA` is `0xe2dae1c0…af7e1` | token + desk API | High | — |
| F3 | Vault `owner()` unchanged | eth_call at pin | High | ownership transfer |
| F4 | `withdraw` still in vault runtime | selector `0xd9caed12` in code | High | new deploy |
| F5 | `exec` / `distribute` present | bytecode scan at pin | High | new deploy |
| F6 | Vault unverified | explorers (prior + ongoing) | High | verification |
| F7 | Distributions still accruing | `/api/distributions` 253 rounds | High | API change |
| F8 | Latest distribute matches API | 5 receipts sum $477.820844 | High | — |
| F9 | Vault ~12.03% `$AA` | balanceOf at pin | High | transfers |
| F10 | `$AA` is Pons v2 launcher token | prior Blockscout name/creator | High | — |
| F11 | Holder-sized exit depth OK | **not obtained** | — | — |
| F12 | Launch cohort clean | **not investigated** | — | — |
| F13 | Owner never withdrew treasury | **not proven** | — | — |

---

## 10. Bottom line

**Still a real desk. Still a real keeper. Still real USDG payouts — larger book than on 2026-09-05.**  
**Still not trustless. Soft-rug capable by design (owner withdraw).**  
**NO-GO under a rug-resistance requirement**; acceptable only as **operator-dependent speculative yield** on `$AA`, with custody and LP/inventory risk the distribution feed alone will not show.

### Coverage limits (not passes)
1. Pinned holder-sized `$AA` sell quotes  
2. SIZE-pattern launch-cohort accounting  
3. Full owner-withdraw history / fee→inventory conservation  
4. Fresh Blockscout top-holders page (CF)  
5. Exhaustive side-pool LP beyond API bands  

---

## Appendix A — Key links

- Desk: https://www.arbitrageape.app/
- Docs: https://www.arbitrageape.app/docs
- Status API: https://www.arbitrageape.app/api/status
- Distributions API: https://www.arbitrageape.app/api/distributions
- Vault: https://robinhoodchain.blockscout.com/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98
- `$AA`: https://robinhoodchain.blockscout.com/token/0xe2dae1c072b9f66bed999873b94798b2152af7e1
- Public copy: https://github.com/adidogCEO/Robinhood-Research/blob/main/Arbitrage-Ape.md

## Appendix B — Live desk status (embedded)

```json
{
  "mode": "live",
  "feeToken": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
  "fund": "0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98",
  "ethBalance": "1.5927402518827611",
  "ethUsd": 2490.98276109,
  "ethFloatTargetUsd": 3000,
  "usdgBalance": 60742.094358,
  "realizedProfitUsd": 118747.92360513941,
  "operatorPnlUsd": -3283.014486199645,
  "lpPrincipalPnlUsd": -98817.34764841342,
  "distributedUsd": 103530.98157799993,
  "reserveUsd": 14370.024580832274,
  "distributionShareBps": 7500,
  "minDistributionUsd": 300,
  "pendingProfitUsd": 846.9174463071977,
  "lastDistributionAt": 1788809834916,
  "nextDistributionAt": 1788810734916,
  "lastScanAt": 1788810601645,
  "lastSurveyAt": 1788749588387,
  "referenceAt": 1788810601292,
  "marketOpen": true,
  "thresholdBps": 2500,
  "pools": {
    "v3": 387,
    "v4": 7836
  },
  "instruments": 194,
  "eligible": 92,
  "feed": {
    "live": true,
    "lastEventAt": 1788810582210,
    "events": 21188,
    "since": 1788810555335
  },
  "backfill": {
    "head": "57096258",
    "v3": "57096259",
    "v4": "57096259",
    "caughtUp": true,
    "at": 1788810551922
  },
  "feeFlow": {
    "escrowUsd": 0,
    "claimedUsd": 227563.3077810002,
    "lastClaimAt": 1788810491084,
    "phase": 2,
    "recipientIsVault": true,
    "at": 1788810491288
  },
  "token": {
    "priceUsd": 0.003555004227908025,
    "mcapUsd": 3495769.4038454993,
    "supply": 983337622.049079,
    "at": 1788810597501
  },
  "burn": {
    "qty": 16680435.532890057,
    "usd": 18756.54249617989,
    "count": 85,
    "lastAt": 1788807494065,
    "lastTx": "0x79aea4ff2c1ff6f176ba2c9ea13a4e504c97a498bf09559b95f7ea1be451bddb",
    "supply": 983319564.467149
  },
  "lp": {
    "enabled": true,
    "bands": [
      {
        "symbol": "RSTR",
        "band": "RSTR",
        "pool": "0xe809802ee8fa68bae3a2e102a47d2b5a7a5d23ecf2c562c720aba0d9b348f0f1",
        "protocol": "v4",
        "capitalUsd": 6000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 3500,
        "hooked": false,
        "position": {
          "tokenId": "2125592",
          "lowerUsd": 0.0010947201721804812,
          "upperUsd": 0.002204418559794443,
          "depositUsd": 2646.112173,
          "depositQty": 1770604.0954947,
          "depositStockCostUsd": 2553.8089024483625,
          "depositQuoteQty": 0,
          "stockQty": 1410198.4101942428,
          "cashUsd": 3221.6290994808573,
          "quoteQty": 3221.6290994808573,
          "valueUsd": 5512.918313188377,
          "feesUnclaimedUsd": 44.93514694428705,
          "feesCollectedUsd": 1024.334006044927,
          "inRange": true,
          "openedAt": 1788807939671,
          "txHash": "0x33012b3f8e88aff436e036a441d7057738ff64416807f7edf636eb5ed7a61362"
        },
        "spotUsd": 0.0016517068909057053,
        "refUsd": 0.0016247991751684888,
        "feesCollectedUsd": 2292.1752595313874,
        "lastAction": null,
        "at": 1788810601998
      },
      {
        "symbol": "ORBIO",
        "band": "ORBIO",
        "pool": "0x4b83b47e62e9a5986a7badf9d2369911034a01c079532ede3c2819f332ac830a",
        "protocol": "v4",
        "capitalUsd": 6000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 3500,
        "hooked": false,
        "position": {
          "tokenId": "2122274",
          "lowerUsd": 0.01892258703211098,
          "upperUsd": 0.02822861813739592,
          "depositUsd": 2700,
          "depositQty": 119612.59343298877,
          "depositStockCostUsd": 3195.875143907136,
          "depositQuoteQty": 0,
          "stockQty": 109705.0609741903,
          "cashUsd": 2930.302618474567,
          "quoteQty": 2930.302618474567,
          "valueUsd": 5500.896695341262,
          "feesUnclaimedUsd": 25.593938031403415,
          "feesCollectedUsd": 407.8327564416887,
          "inRange": true,
          "openedAt": 1788803666064,
          "txHash": "0xe5fad534f5f255cf2827f2329697a6697e2430c6674ff04a160588a333ac1209"
        },
        "spotUsd": 0.02343186407299353,
        "refUsd": 0.02343186407299353,
        "feesCollectedUsd": 2289.896994061637,
        "lastAction": null,
        "at": 1788810602759
      },
      {
        "symbol": "PERPSHOOD",
        "band": "PERPSHOOD",
        "pool": "0x5aaaa68d5731cde4ed3c9fd72903e125c204dc428777ab568364e0c29a7d9937",
        "protocol": "v4",
        "capitalUsd": 6000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 3500,
        "hooked": false,
        "position": {
          "tokenId": "2118344",
          "lowerUsd": 0.0038207100574393876,
          "upperUsd": 0.005699715638360709,
          "depositUsd": 2586.831331,
          "depositQty": 566854.8417486,
          "depositStockCostUsd": 2745.4631414477976,
          "depositQuoteQty": 0,
          "stockQty": 749154.3552172722,
          "cashUsd": 1763.3772083735385,
          "quoteQty": 1763.3772083735385,
          "valueUsd": 5112.533160764488,
          "feesUnclaimedUsd": 48.152971593890356,
          "feesCollectedUsd": 959.8588207824257,
          "inRange": true,
          "openedAt": 1788799194836,
          "txHash": "0xee1dc81699beeccfea4d8d86e0fa8644f0c99ad19a477314d02a0dd5b24d49f0"
        },
        "spotUsd": 0.004381596569182787,
        "refUsd": 0.004470581968944992,
        "feesCollectedUsd": 3163.280909769805,
        "lastAction": null,
        "at": 1788810589083
      },
      {
        "symbol": "AA",
        "band": "AA",
        "pool": "0xe2156fc6454acef70a7763cf0f90e207c182880388ab3d381b84dc5d2e91b1bf",
        "protocol": "v4",
        "capitalUsd": 20000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 3500,
        "hooked": false,
        "position": {
          "tokenId": "2127104",
          "lowerUsd": 0.0024362471030970516,
          "upperUsd": 0.005699715638360709,
          "depositUsd": 8597.561471,
          "depositQty": 2423031.4783655996,
          "depositStockCostUsd": 0,
          "depositQuoteQty": 0,
          "stockQty": 2648893.2603197517,
          "cashUsd": 7778.668731239769,
          "quoteQty": 7778.668731239769,
          "valueUsd": 8597.561471,
          "feesUnclaimedUsd": 30.027433005241093,
          "feesCollectedUsd": 327.41145973758955,
          "inRange": true,
          "openedAt": 1788810211905,
          "txHash": "0xe7af34f8cb780738a6402872f5f80ef20136ef768ec99f43b97fc16770af9a16"
        },
        "spotUsd": 0.003560841305887195,
        "refUsd": 0.0035481012156911287,
        "feesCollectedUsd": 55218.17378061839,
        "lastAction": null,
        "at": 1788810589938
      }
    ],
    "feesCollectedUsd": 149749.25599749645,
    "at": 1788810602760
  }
}
```

## Appendix C — Compact summary (embedded)

```json
{
  "generatedAt": "2026-09-07T19:59:11.859120+00:00",
  "reportVersion": "v4-refresh",
  "pin": {
    "block_number": 57099428,
    "block_hash": "0x80e6ce7ba61459ef97604844b2c75a7123a183531275b0bb53fa1d5b2366f15c",
    "utc": "2026-09-07T19:54:31Z",
    "chain_id": 4663
  },
  "rounds": 253,
  "onchainTxs": 978,
  "multiTxRounds": 203,
  "sumListUsd": 103355.870787,
  "status": {
    "mode": "live",
    "fund": "0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98",
    "feeToken": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
    "distributedUsd": 103530.98157799993,
    "realizedProfitUsd": 118747.92360513941,
    "pendingProfitUsd": 846.9174463071977,
    "usdgBalance": 60742.094358,
    "ethBalance": "1.5927402518827611",
    "lpPrincipalPnlUsd": -98817.34764841342,
    "operatorPnlUsd": -3283.014486199645
  },
  "feeFlow": {
    "escrowUsd": 0,
    "claimedUsd": 227563.3077810002,
    "lastClaimAt": 1788810491084,
    "phase": 2,
    "recipientIsVault": true,
    "at": 1788810491288
  },
  "token": {
    "priceUsd": 0.003555004227908025,
    "mcapUsd": 3495769.4038454993,
    "supply": 983337622.049079,
    "at": 1788810597501
  },
  "onchain_vault": {
    "aa": 120307567.25533798,
    "aa_pct_supply": 12.030756725533799,
    "usdg": 60072.736679,
    "eth": 1.592740251882761,
    "owner": "0x12b46b7746af4a902fd55198f09ecfd8c4d42956",
    "keeper": "0xdef933cfaeb2a2af516df1eaa2101b00b7f77af6"
  }
}
```

## Appendix D — Full distribution history (embedded)

All **253** payout rounds from `GET https://www.arbitrageape.app/api/distributions` at this report generation.

```json
[
  {
    "at": 1788527950582,
    "totalUsd": 0.8666,
    "holders": 33,
    "realizedCumUsd": 1.190486,
    "txHash": "0x567e40ea130d06d74192387b2fe240c659dc2b70f50c51ad39b8ccdcf14a2f51"
  },
  {
    "at": 1788529482707,
    "totalUsd": 2.096236,
    "holders": 53,
    "realizedCumUsd": 3.329867,
    "txHash": "0xfa7c144df39842ac6e65548ee75dcc2ef0cbe5ff51c9d094e41dd488211e5a7b"
  },
  {
    "at": 1788530046297,
    "totalUsd": 1.580583,
    "holders": 54,
    "realizedCumUsd": 6.765765,
    "txHash": "0xa86b94cce1cfb8eccb4834fe2be369a515f159e55e9d9dab573ffc12986c1237"
  },
  {
    "at": 1788530230488,
    "totalUsd": 1.850904,
    "holders": 60,
    "realizedCumUsd": 6.765765,
    "txHash": "0x97e11050773da7d3f59f09b504adabedb6eb4776c2fdbca3cba650682acd2cf0"
  },
  {
    "at": 1788530560580,
    "totalUsd": 0.851661,
    "holders": 37,
    "realizedCumUsd": 7.637293,
    "txHash": "0xcb868ec6ce8bf46844a4fb0ac09231bac3310f0f11afc1194a35ad7de94dbb98"
  },
  {
    "at": 1788533563001,
    "totalUsd": 11.287618,
    "holders": 151,
    "realizedCumUsd": 19.22080529639458,
    "txHash": "0x06e0e5ad8741adbad07fd3a1a2f88a7bfbb824dba96aa90b6b29d993099ce1b1"
  },
  {
    "at": 1788534249214,
    "totalUsd": 0.736891,
    "holders": 36,
    "realizedCumUsd": 19.73796229639458,
    "txHash": "0xb928c3a694ba417c242d6a54e14b7d0cf569ead97d49f915b3a34ef5baae2b0d"
  },
  {
    "at": 1788534545432,
    "totalUsd": 0.634853,
    "holders": 33,
    "realizedCumUsd": 20.29149029639458,
    "txHash": "0x4a1de751c3b135dc235adfe63a709c3b53a7fa703f226efb8b0e3d5bbfe38cd2"
  },
  {
    "at": 1788535476408,
    "totalUsd": 0.787033,
    "holders": 38,
    "realizedCumUsd": 21.11334929639458,
    "txHash": "0x33cc3025a3270b6ae3043a5bb2f5fae9755cf1d6da1946c4e9e76ccbf7a0d6bf"
  },
  {
    "at": 1788536272247,
    "totalUsd": 0.617432,
    "holders": 33,
    "realizedCumUsd": 21.72469629639458,
    "txHash": "0x5e2c6660ea58a4e834a80f00812f2c6e74a6e8a3b99769db396a5170c7a1b59d"
  },
  {
    "at": 1788537685625,
    "totalUsd": 1.006611,
    "holders": 41,
    "realizedCumUsd": 22.77544329639458,
    "txHash": "0x1cf199d2278fd9bab341e361d39aafa06ebd85ded99b1d96b0086cfb1d6021ca"
  },
  {
    "at": 1788538585542,
    "totalUsd": 2.671577,
    "holders": 70,
    "realizedCumUsd": 25.570712296394582,
    "txHash": "0xecb1b71a0dd13d47b0275de40e78f85fab449c758179cd30e2632c9e2909d0e2"
  },
  {
    "at": 1788539498684,
    "totalUsd": 0.823394,
    "holders": 36,
    "realizedCumUsd": 26.24423829639458,
    "txHash": "0x10647b87973a80042431d971be7381f6b6b109ecf21696313dabba36c3dc0bdc"
  },
  {
    "at": 1788540350174,
    "totalUsd": 25.118646,
    "holders": 199,
    "realizedCumUsd": 51.59118529639458,
    "txHash": "0x2c46ef542bf4082bc4a09576bc659470b4dea2d919cc19eff57df902794b9ecf"
  },
  {
    "at": 1788545173764,
    "totalUsd": 19.960073,
    "holders": 194,
    "realizedCumUsd": 71.50510229639458,
    "txHash": "0x50de1074a1c898b5edba12a77bbe1f7d182218f25aed201c3537fdb8c0c3d15c"
  },
  {
    "at": 1788546071232,
    "totalUsd": 0.561498,
    "holders": 27,
    "realizedCumUsd": 71.90273329639459,
    "txHash": "0xdf4bfb64f421afd2824699e9b8c89861e592ceac84089871ea7d20701b30976f"
  },
  {
    "at": 1788546757593,
    "totalUsd": 20.557595,
    "holders": 196,
    "realizedCumUsd": 92.6269182963946,
    "txHash": "0xf65f546c383d85c1ab49db7b340633cac490cbbff52557596bbec586e9a67caa"
  },
  {
    "at": 1788547654212,
    "totalUsd": 1.161306,
    "holders": 48,
    "realizedCumUsd": 93.68168229639461,
    "txHash": "0x280a2d8a70583b9f94fae14b99b3e24ac5f7dd557c9dba54965dfeb0b82558f1"
  },
  {
    "at": 1788548244182,
    "totalUsd": 1.470659,
    "holders": 56,
    "realizedCumUsd": 95.17732629639461,
    "txHash": "0x596b516e44837c50360f25795aeba80799a88930925cf19623a662ad59502271"
  },
  {
    "at": 1788548753072,
    "totalUsd": 1.696331,
    "holders": 59,
    "realizedCumUsd": 96.89172229639462,
    "txHash": "0x5c46b45e9008e66663d0189ae496535726030f4d6da568872044cbe59c25af9d"
  },
  {
    "at": 1788549651295,
    "totalUsd": 1.178022,
    "holders": 46,
    "realizedCumUsd": 98.02757829639462,
    "txHash": "0xf2bec8c79b38947ef16803cae24f16bf667148610e9e4cba01b2570e19c53eb3"
  },
  {
    "at": 1788550588757,
    "totalUsd": 5.090191,
    "holders": 107,
    "realizedCumUsd": 103.23524029639462,
    "txHash": "0x36633554c76b166a680797a91a4c7da108bcb6955efb911e787555ad8f4ecdb5"
  },
  {
    "at": 1788551489328,
    "totalUsd": 6.374034,
    "holders": 120,
    "realizedCumUsd": 109.62978429639463,
    "txHash": "0x18bf37a5792521265ec99ec71a265e52199b67456fee4958841c2f2634106d3f"
  },
  {
    "at": 1788552387963,
    "totalUsd": 21.151341,
    "holders": 202,
    "realizedCumUsd": 130.73744829639466,
    "txHash": "0x178cf347afe9c5767a095e012f42f6cc9eeb240e531406fa3ae89ca291d7dcf2"
  },
  {
    "at": 1788552821963,
    "totalUsd": 1.916286,
    "holders": 65,
    "realizedCumUsd": 132.63956929639465,
    "txHash": "0xf2abfcf063fac6a9bad31e79b28d41371562b929cc81159e2a80b14ea6af275e"
  },
  {
    "at": 1788553715641,
    "totalUsd": 4.000896,
    "holders": 99,
    "realizedCumUsd": 136.68355629639464,
    "txHash": "0x52cffb8d56f70898e70029978e3f137921b116494c2de5e83e2e91a8142b0bf6"
  },
  {
    "at": 1788554617621,
    "totalUsd": 13.844468,
    "holders": 172,
    "realizedCumUsd": 150.5853722963946,
    "txHash": "0x6586abc3f479876b417aba1bb2090ca4139fcec5bc352024fc32380bd1310db4"
  },
  {
    "at": 1788555520753,
    "totalUsd": 18.417207,
    "holders": 237,
    "realizedCumUsd": 169.65433229639459,
    "txHash": "0x1bd029882404099a3b1b2ecc7f12677b9dc93ec29db574214e855d8c0d6553b7"
  },
  {
    "at": 1788556136672,
    "totalUsd": 2.653773,
    "holders": 88,
    "realizedCumUsd": 171.95257029639458,
    "txHash": "0x6112bfff93569941d5c2ef82881fb2547b6690d614f3efb167e226eaa94ca115"
  },
  {
    "at": 1788556303871,
    "totalUsd": 1.660253,
    "holders": 59,
    "realizedCumUsd": 173.5522822963946,
    "txHash": "0x8df02fc7958ae89d2b657188c3545169ac73879a18490d5fc3c64688bf0f1b65"
  },
  {
    "at": 1788562486854,
    "totalUsd": 1.955239,
    "holders": 64,
    "realizedCumUsd": 231.4812462963946,
    "txHash": "0x5b8b5381d504ca8cb48a6cab9d4cfd383355c7feb46aad1dbce259a5c91ad238"
  },
  {
    "at": 1788562714283,
    "totalUsd": 97.384906,
    "holders": 455,
    "realizedCumUsd": 274.2772782963946,
    "txHash": "0x2208f8d29f1d2719619f3606b73a8aafe5cd361a5f113e6de7bdaee2c01988b6"
  },
  {
    "at": 1788563600829,
    "totalUsd": 4.317297,
    "holders": 113,
    "realizedCumUsd": 277.26797429639464,
    "txHash": "0xb5b690e323efcc1de120e0722a759945421f966dc909d5e704149f0757d7712b"
  },
  {
    "at": 1788564501081,
    "totalUsd": 5.681059,
    "holders": 135,
    "realizedCumUsd": 282.97701029639467,
    "txHash": "0xd77b2ba1ed6c3820e9571f1c4de4f6b2445634af3497703ddde8a074685ee795"
  },
  {
    "at": 1788565153496,
    "totalUsd": 15.665388,
    "holders": 215,
    "realizedCumUsd": 299.4533312963947,
    "txHash": "0x55d54a8f2739d5ab2992c3fc2a0dff59a18b694ed8beaf3cd4d6aa25731cdc4b"
  },
  {
    "at": 1788565339875,
    "totalUsd": 1.520186,
    "holders": 56,
    "realizedCumUsd": 299.9502352963947,
    "txHash": "0x98f8ab898fd4b320a7f0e7c8df4e6f3fc8fd3dee63bc6cac3a0244447f53e1b9"
  },
  {
    "at": 1788565496292,
    "totalUsd": 0.677159,
    "holders": 35,
    "realizedCumUsd": 300.42632929639467,
    "txHash": "0x931060d13af6bcc1df6406303bb6855e5ac34f40ad4f60316bf2b975e9031e06"
  },
  {
    "at": 1788565650192,
    "totalUsd": 0.760574,
    "holders": 39,
    "realizedCumUsd": 301.1829882963947,
    "txHash": "0xf68e19c754d631ed4d0fa4fa6cbeaba69991cc2e474bd57d0784eb0b7657f4c0"
  },
  {
    "at": 1788566083199,
    "totalUsd": 1.095299,
    "holders": 47,
    "realizedCumUsd": 302.3601672963947,
    "txHash": "0x8b02cd92d4657fd4999889f3770fd599a4bab69fdf334138c54b47f6bd16c115"
  },
  {
    "at": 1788566271041,
    "totalUsd": 28.178153,
    "holders": 272,
    "realizedCumUsd": 331.14712629639473,
    "txHash": "0x9f649fc72ae52f89f8f5f1d276d7b7e318fa908cb3a9c7ac2711d283bbb94167"
  },
  {
    "at": 1788566386880,
    "totalUsd": 1.546064,
    "holders": 55,
    "realizedCumUsd": 362.23110329639474,
    "txHash": "0x9d1f9a550a7569def77d4b78396e850af15a1b9114deb90795892a3c6766d854"
  },
  {
    "at": 1788566604176,
    "totalUsd": 71.647593,
    "holders": 384,
    "realizedCumUsd": 504.95721829639484,
    "txHash": "0x811c4c89ee3819520fc41a1f2b1267de2707da7fd78f34976e050332205a493e"
  },
  {
    "at": 1788566846544,
    "totalUsd": 115.028904,
    "holders": 452,
    "realizedCumUsd": 519.1456572963948,
    "txHash": "0x6c850ee43842d2ecf3056eeb228a07ec0c36c0b8801dd755dcb75c348d776522"
  },
  {
    "at": 1788567789956,
    "totalUsd": 442.061354,
    "holders": 600,
    "realizedCumUsd": 960.8948882963946,
    "txHash": "0x2589b15dc24704d739ef2278b40954401e68cf79f09261257b26cbb441a9e232"
  },
  {
    "at": 1788567920856,
    "totalUsd": 90.372049,
    "holders": 408,
    "realizedCumUsd": 1056.9179582963948,
    "txHash": "0x251c29281fa68f983e5901a61ca486cc8a7d2d9f0bee533df2afde981c478e10"
  },
  {
    "at": 1788568072223,
    "totalUsd": 7.005505,
    "holders": 140,
    "realizedCumUsd": 1058.5233302963948,
    "txHash": "0xb4fddf21e9bd67495786c8e4f9c4c0282999f47dc6a8aa6eeb2ebac8828c8170"
  },
  {
    "at": 1788568918243,
    "totalUsd": 80.788084,
    "holders": 396,
    "realizedCumUsd": 1141.3465441678625,
    "txHash": "0x9eb454c6e1e2978020399a1ccc75208eae580e22dbd825c1e6c4fc3995f3d3e2,0xbb350398f0eda6ed9cf3ad294351d90d97fee89bcf70659bd670fff8617a956e"
  },
  {
    "at": 1788569830297,
    "totalUsd": 339.432743,
    "holders": 605,
    "realizedCumUsd": 1478.6162721678625,
    "txHash": "0x7d8e5c8a6e44477571d1ad5e1bf4946ef2b43b6f6164493bcea195c2a569d232,0x62ff6b0e88d6c87464a3a067fd057018d8a0f4c6f61b24afdda52b1ac7d42899,0x8840219449beee31a337f9a545a8474c5710961f063a046c9927c802f51a5f5d,0xdf00ecbfb5dcf35192216b17aa5c9f2fcb1a718d99b575e43306b811951b0454"
  },
  {
    "at": 1788569913519,
    "totalUsd": 129.682861,
    "holders": 491,
    "realizedCumUsd": 1608.4713181678626,
    "txHash": "0x3e1407472070e5609d057fcbcfe4b173d674e7bf753870708924fc656135ed90,0xc5ed7d3e68a18b1df705f247dccc226644b245745edde05334192a595bba705f,0x7e8b6c6b14589a401e0eca6d9a4ab95c9dc888b53c4774ff4f8a0ac972829793"
  },
  {
    "at": 1788570810029,
    "totalUsd": 231.556895,
    "holders": 567,
    "realizedCumUsd": 1839.9431205914134,
    "txHash": "0xf965bae5017f838aa1225c9574e48728e1c01c93b662c7e23a03fb8ed5c5b432,0x5525f9eabff55cb560d543580bdf38cfc8b8f3e35b0055e13fadc51463d57619,0x4d2bed9f11ce456010e3f119f2d6ea96f103713a7bfd9d851a003112c71658d6"
  },
  {
    "at": 1788571650322,
    "totalUsd": 91.717421,
    "holders": 443,
    "realizedCumUsd": 1938.660314591413,
    "txHash": "0x43bcb2e6b190ad2125a5b6967c24ca7e23d4e89f82fffeac4b243ea1827970ca,0x440d74f625547cb0c7ea804b33aae62297cede7ffd40dc9f10e147fc5d14614b,0xf9084f1180434956a46f292ced2a670b80e396b7e5f4849b5b2b21208ff51a0b"
  },
  {
    "at": 1788571835436,
    "totalUsd": 114.614536,
    "holders": 484,
    "realizedCumUsd": 2052.419302591413,
    "txHash": "0x49df6fa72fa28237486ec661690825180c978419c81b8c64248e8b30c39923b4,0x9f063f405ac083e54d5c8def5140584fed49e7d263004ae9ebe44d1eddbb67e4,0x66b4b8ce75d9f46b543e6e0d3bb989d26f5e24819da9dbbd87055ae3d6f51646"
  },
  {
    "at": 1788571923273,
    "totalUsd": 82.296052,
    "holders": 432,
    "realizedCumUsd": 2128.9090455914125,
    "txHash": "0x7ac3812a9375080013b88fd47e6910e94ff16fb624eb8468a9b2d625278d58e8,0xce400e0f2d86a99f1d5833bebc5d2ee8cf576e1da361460900ae33d553950532,0xf0332a397f03620d7fb08b679adde997244480110df68d31c4c755005f525f6c"
  },
  {
    "at": 1788571998491,
    "totalUsd": 34.140929,
    "holders": 306,
    "realizedCumUsd": 2179.529958591412,
    "txHash": "0xaaeb028f75839cbdd040015ab22e95600d6b170667983cab05229021c7ccbf9e,0x66b888b5e2434fa32ba4bff6234d5158541fdb740fc9e080e54842a0643ce92f"
  },
  {
    "at": 1788572603170,
    "totalUsd": 176.741496,
    "holders": 539,
    "realizedCumUsd": 2369.749022669569,
    "txHash": "0x9a61adcc698c8ba4aad6c0dd6a52db38fc924bf9b6ef018a2e485632b6d228a6,0xc86b8bcd5bbbe7b36d7d6c79deab4ed7128672de15b1aa6af7f2bc9444f87e17,0x15b3342c6f0bd23a58c9d614a595e4f899cb870c64071ab3c6e541f8da9d0033"
  },
  {
    "at": 1788572686491,
    "totalUsd": 46.530364,
    "holders": 348,
    "realizedCumUsd": 2387.127055669569,
    "txHash": "0xcec39c56088e900f3a689e1a953317019403009346b8a92ea1da037058d54dab,0x125022c1a7595bc3643c37b3c6cea3ea612086fb28b2f6e9d22ada311584d062"
  },
  {
    "at": 1788572788631,
    "totalUsd": 1.31104,
    "holders": 52,
    "realizedCumUsd": 2428.717818669569,
    "txHash": "0x07e9470f43263a87dfaf0caee0b832596037856fd3d8abc61a7566f40af1ee51"
  },
  {
    "at": 1788573578440,
    "totalUsd": 57.535179,
    "holders": 384,
    "realizedCumUsd": 2495.6451658938477,
    "txHash": "0xf756f10dc4137d38857fcb054d8d7996a78f28e250437024cd5fb2a3bf0f7d9d,0x2284aeae5c8e2d1c63b5d86aa373c9d3ef73b40e804a814a1ce008e447cb0da5"
  },
  {
    "at": 1788573661127,
    "totalUsd": 331.471083,
    "holders": 615,
    "realizedCumUsd": 2776.2204388938476,
    "txHash": "0x3c6e2638482461a111550234c3dba97999685f90ba8bb4d2322f8ef71d53930e,0x1a9400b68128791c3e93776a5f97473977b57dbac83506e3b51aafb595f30894,0x50fe09b83520ae9a46c9ae376804de398bd0203ec5dcb0061b64b8c889fcd7e5,0x25d1993d9b2b6a0d14bd12d90434a57582cb483c0fe01beb853fd60de478e265"
  },
  {
    "at": 1788573913856,
    "totalUsd": 470.411894,
    "holders": 650,
    "realizedCumUsd": 3246.557914893847,
    "txHash": "0x4176de34bea83b35da403d52cb057952c4f4696b75216e5c43c4c1b84af523d6,0xeab42fc95c65f274b0ec7ffc4ad6d6ec42999da8b03ffad1407afcbc78aebfe9,0x6aaf8b4e8073ffcd9508aedb6ab7906587f170b7caca137cb85aa66378eb060d,0x4a255e34b821dcaa38abdf412cb986cea3a08a814e83a3a011c238fc0d0c3dc2"
  },
  {
    "at": 1788574043097,
    "totalUsd": 28.233265,
    "holders": 288,
    "realizedCumUsd": 3275.311537893847,
    "txHash": "0xc7d349fbe4a281983f79570347fa4835d3c44e26c8ca3c16ad1a6423005b400d,0x52b7e47e1d40773cb7b28dddc1f0553b7eb78a0947a8192941145f3aa23c9f23"
  },
  {
    "at": 1788575480857,
    "totalUsd": 732.329373,
    "holders": 703,
    "realizedCumUsd": 4008.4409047101535,
    "txHash": "0x79ab7ddfd7b831e2d45d485c620180dd107438c5ecadbeb9a66e789c417c2af3,0xb94ee106a30fc59b42502c8bd33a0678eef3a1923a2c24ab5075dbdd4cb5ed00,0x51e016a0d41bd5e6cfc1733d9746cd438f7d1e1b001f93afd672d667b4f8e535,0xb7fae123865e141a772885b542466904b7e5fa9e81163410133c4e3dc05c5798"
  },
  {
    "at": 1788575915645,
    "totalUsd": 27.22083,
    "holders": 281,
    "realizedCumUsd": 4034.8780537101534,
    "txHash": "0x73dd034351b85755940227fef2b4478c80be63ac139693a98a942a0ea247747b,0xe385688b13dd432d5c22f82e7a51afc7a765995d74ba7b93cc801bab81fe85da"
  },
  {
    "at": 1788576266711,
    "totalUsd": 82.381579,
    "holders": 432,
    "realizedCumUsd": 4118.339118552331,
    "txHash": "0x1a305eff993798de7ec40e5ea1b01dbbb0190f4907ebf2438573ceb0ce05cc5c,0xf8f0d4c9a5121220417638df26705290e064dcae0a28f5c6c993a4d07d8a4e52,0x58499b8b98fb6d2cce8748b578ef974eb1e2f6542b4c9430aef8c7a8c9eae6f6"
  },
  {
    "at": 1788576809861,
    "totalUsd": 387.268581,
    "holders": 626,
    "realizedCumUsd": 4505.026506925274,
    "txHash": "0xa6e2389d6bf11c6e8388942e84bea4e4ab23c523c69a5db0b6e530a1b63712e2,0x93f36d4be208e8e85fb4a056bcbc4d8efa77880abf842cfb971960f43f752e61,0xd57a0244948fcb7458fbde1fcad9b0e0c4edb7cd08f7a6744e112db834e05f40,0xcb03af5929144aaebf71f33c7549bf88b4c436de14ec02b96d4e532474f8e4a4"
  },
  {
    "at": 1788577124004,
    "totalUsd": 96.487083,
    "holders": 462,
    "realizedCumUsd": 4601.470584925273,
    "txHash": "0x21095142ab53952d15e5ef5ff66d7c9da50f38be544802cba029888737204155,0x987d28db3c1540cd6bfbc03dbb22e32d0c51b57aa4a218029700d33c34400227,0x734251f2a5b64c3996cd37accb079e841d9a4fbeeb7414cda807f7c8ddf48d67"
  },
  {
    "at": 1788577630477,
    "totalUsd": 94.614516,
    "holders": 451,
    "realizedCumUsd": 4695.500217925273,
    "txHash": "0xb7003a9b0e897030bc5a282772898cfe3f87441c6ac2b1104052e3a0e9fef1cc,0x9926355f974f0e3f0f6ff2bf99468cd846902b132a1f849d9e2c9654504f52b0,0xef70b537c55f815b960e66210f3cdf83e5bd0b9f38c7465261abc40d7790d8bb"
  },
  {
    "at": 1788578078569,
    "totalUsd": 301.500718,
    "holders": 606,
    "realizedCumUsd": 4997.36318270526,
    "txHash": "0x61e72b17d63e7a272fba30ab566a8573e207fa6b6f5a78bea5df39896a5f8145,0x3b3f1bdbe09ba1b3bf22b1c0fdbb47474b596ef10c217dcb70f1a18ecf50aea6,0x8dd69cd7536795660a6c70d5ea30c9ec638fc0b87f2a078ea1e287a8afff5520,0xb6cb89ceb8c289b4e53b6d32591d9316d357f1824d1fdc840f4f897646df83a5"
  },
  {
    "at": 1788578431222,
    "totalUsd": 41.888048,
    "holders": 351,
    "realizedCumUsd": 5038.902937982995,
    "txHash": "0x61b81cf342f3a65b0b4ddfa5d8f087ea469e259c3b192a7d208a46f605ddbbbd,0xc5a771dd4c434e4a2453a1773bd29b2987a64bd8fdbdd213983a32e0fff711d5"
  },
  {
    "at": 1788578696412,
    "totalUsd": 32.86455,
    "holders": 305,
    "realizedCumUsd": 5071.9166176130675,
    "txHash": "0x599bcf6a0c0e244961280ae3bc554f9c5c3782554c83c5f6d3a65d02adc30181,0x10f7092befbebde35cedde9aae557853d467cadc40383cb0b6e99060c40bd1bc"
  },
  {
    "at": 1788578820689,
    "totalUsd": 18.540206,
    "holders": 247,
    "realizedCumUsd": 5136.141206613068,
    "txHash": "0x26b15ec390882964a4aad7b24e75a498f94d5695d773b36795cce0190dafcef5,0xea19f0b6b79cfc906a8d302dff8747488249450485df3e9d8281e4f554fed066"
  },
  {
    "at": 1788579513054,
    "totalUsd": 646.418741,
    "holders": 703,
    "realizedCumUsd": 5736.2177056130695,
    "txHash": "0xb367aee7903a979e447f0413063f9278b4113ee144fa8521ae6e6374c3d8bd3d,0x303640b6967f0f4ac4c6ae7200b8dd6c74aa273d29ea50b9122304a43726cc26,0xee64764a83b30ba3feef3e8f326aea080462d4693b51d14a2099f68a916ba95f,0xbaed9fc2e690038ae15464760aa2eacd186d1eb38b9947129cfa4061f5d7b100"
  },
  {
    "at": 1788579712992,
    "totalUsd": 46.429577,
    "holders": 358,
    "realizedCumUsd": 5783.26919261307,
    "txHash": "0x3f68c241f483a428398d6b1d9948c68a7f9a391ea12d61e5c83f78dee4eb61a1,0x6aea773e4a1bf4be3c06afec8bf9deb3096dc35be9ab55ba8090a80056bc60d8"
  },
  {
    "at": 1788580124786,
    "totalUsd": 123.921224,
    "holders": 500,
    "realizedCumUsd": 5971.56776661307,
    "txHash": "0x9708a931518d5ef4c6e77e18b2a1796a22b409624a8da0065158a509cc1a3051,0x579e9a38ee581db42abe5056d4a1dd9656d79663be3c555f1dd6ecc470415d6f,0x97b5bbbf74de385dd9c3b2f4f8f4781c852565551eb1612dbe28aac3686b8226"
  },
  {
    "at": 1788581020409,
    "totalUsd": 225.54528,
    "holders": 605,
    "realizedCumUsd": 6132.467178613069,
    "txHash": "0xbb1f745e8183f338038b0f1b3f00dc214220e8fa7b67ecb2016ee827953a2f20,0x91ddaef94b4402c599bcdd7fabb43c61e5e4b8c1985954a330dde229fe7e3dd0,0xfe753a55dc40226b11a3460880af69e5d087d605652c8505ca56b1954d5e6658,0xdab6af3920fd6895e28ce6edee5a04f796e0e775a08ff0755f61ac75e497584e"
  },
  {
    "at": 1788581909372,
    "totalUsd": 430.257155,
    "holders": 687,
    "realizedCumUsd": 6562.62705661307,
    "txHash": "0x2dc127597e7e7dc4f6bd76d7893f5d39019719c1869e6ad3ba3dae9f5e791d84,0x2419b7a014f70ae5fccbe45f41e2ae1122eb80b72e5efc24315c2a370ceeffcb,0xf66ff292dcdf575e887207db60bb1b9baa5d1ad6a6f4b6a5deedb519b2bd3fa6,0xc17b1ef5fcc37a726f93bb67f806537775206674314414de6f88a25f87a500e1"
  },
  {
    "at": 1788582292348,
    "totalUsd": 9.429414,
    "holders": 193,
    "realizedCumUsd": 6624.57091361307,
    "txHash": "0x67465ba045a47ce43cae05e5dfda7ced3c1f9e94b7d61c27fbd6195ddc0b1069"
  },
  {
    "at": 1788582776412,
    "totalUsd": 179.715917,
    "holders": 686,
    "realizedCumUsd": 6777.88428761307,
    "txHash": "0x2f9c82952b3943c3c86d71e05b748619ad8f2ba229b48ffa30b7655f2e4138ae,0x145f70f88d96ad767b2b5357a54e23163de147b89f761628318958b26e2defd4,0x6bc088daa2cd365bf976db9c4ff6102361096737eea161b956030cd83d862e96,0x2e724403bffa307550570e9c14ff68adedf16104c0dc02d2f0d89bd3dcfbf80e"
  },
  {
    "at": 1788582883473,
    "totalUsd": 73.854553,
    "holders": 494,
    "realizedCumUsd": 6828.8894386130705,
    "txHash": "0xa0277c908b8ca37d9ba81bd3061150feb406dedd5de68f1b9d32d9b44b69b2f9,0x936f6748a4691610daf38a64b18c4cf0f2f71c67cd53208868e4e33915ccb127,0x3a327a5d2d7cbb385a9238e519cf281deec784c0d5f99c256b12b9f8988b70c4"
  },
  {
    "at": 1788583779300,
    "totalUsd": 141.375443,
    "holders": 682,
    "realizedCumUsd": 6972.880549613072,
    "txHash": "0x40234f14077746029ef73e46b4051a19d6df9ff101958707e613cc9a79d91250,0x5b5d7253f972f36a8ef1291165e3610a59573828b02356dac08bf676318fe017,0x10fca1b20b7b27502cc6ff6c9ab0d806deadad0ab49c0a1c73ff20fbc6204ebd,0xdc9e59d632623e092cf9ea1f2bfb48a90cec249032a58ee1e3e30a55dd956459"
  },
  {
    "at": 1788583913404,
    "totalUsd": 192.731382,
    "holders": 754,
    "realizedCumUsd": 7165.295392613071,
    "txHash": "0x1b47d832c04fc8d3b756bd984fab173b8976ad7058051ee49f177bf4e38146e2,0x5fe98d04ccc36cbb182ca05e2269d28f97f9e3f6b33141888640caed6b2b34d2,0xbd2f02986d1b1f95be9fefcac0f977889bdb06cd95a70a13c3df90208d328af4,0x81b6f525e22bde432b096bc19ef68133f21f0ac62811d8805d36159302d08bfb"
  },
  {
    "at": 1788584608921,
    "totalUsd": 412.688823,
    "holders": 904,
    "realizedCumUsd": 7608.854989613073,
    "txHash": "0x4ff19f503c9fdb3901699d608c2d67e0b87f73732f12fcd71c2f1edbc20e0287,0xf55aae5ba3fbdf587b9e0025f7312717d1ce78e493018c853c1bb06c4c6c12de,0xf31384b3e2602f549a9bb04652130d47796417b021c8d8b1077df6b78d6fa4e4,0xfc9fcbe3cc88c0fc810a590b06741afc0d38d57665698ac9bc3433c8f4c639ef,0x28b93449002bb8e2ce24723b37a5b307456f01a4f51d6bf0858782bdded78bf6"
  },
  {
    "at": 1788584749915,
    "totalUsd": 99.949501,
    "holders": 588,
    "realizedCumUsd": 7699.312557613073,
    "txHash": "0x49bde670f2aa4ae0ce57c27b17f26f00b568e03bed8a268180877a7fd9168447,0xe8c9ac6bbef06b445745172268cd5962b1fa833e5de6d590869979ccc5c09bfe,0x72e5697a338914a57424c7cbe2c8e2704b29b2687bae992867cba8200bda774e"
  },
  {
    "at": 1788584927210,
    "totalUsd": 30.839178,
    "holders": 370,
    "realizedCumUsd": 7802.635200613073,
    "txHash": "0x68af428794ab6e563fd9e3f06993878ae6a22736c4069581ba176b3c476b99db,0xff3751883abe4dae9d17c548966812c1b6c782b0575cd7898a74c6111ff82005"
  },
  {
    "at": 1788585367296,
    "totalUsd": 429.740016,
    "holders": 913,
    "realizedCumUsd": 8133.827631613072,
    "txHash": "0x736eaf0aecc4cbebbd5a7f4254b2540379143da939fac56d5959d973cbe18d5c,0x86477c777e9a17c93e4cb76a673ea33d328b716f9c0157351fcd479140c4f471,0xfffcfe922c2c7cffeb17f940cd3ae1f89dddec06d79ca1940958b1fa1dac6e00,0x7ac63d54aed30b64866286fc35cf6a81f310f1cd3579ec2f1cd95658afb3e536,0x4a109b4efcd998d431d2c9556ded14c54ed8ccf3c1a03dfc0b00074a6491e0e9"
  },
  {
    "at": 1788585525848,
    "totalUsd": 151.653479,
    "holders": 671,
    "realizedCumUsd": 8285.826069613073,
    "txHash": "0x6971c8e41b68a13d762aba5eb28e29ab307b8efe7250fa03a8d9183c92d2e95e,0xdeb73a53815ba2436e2b25ef83e128a265b880494e47072e3b3d5a5a42dea52a,0xdf308809cd1d2ecc87e2c2858c99fa26c4deb3a6273682c4bfb8541f99cf42b2,0xdd2f8b2227f149d7994777e6143dcbddf235e69b2e61c7ae47ff376049a18e47"
  },
  {
    "at": 1788585582840,
    "totalUsd": 148.887805,
    "holders": 669,
    "realizedCumUsd": 8467.327524613072,
    "txHash": "0xf0f7c14cafb1c9b14d81de48382ff779b9fde2ed2a8d171de20acf83f1b4f486,0x8041afee808f16eec61d7d3ee0c9e207fc95b1147959912323e03e4799e7f6d1,0xa2beb062f251400e7e9ec2fd90931f3a52a40e9cee6e11f2c7b429da67a5f297,0xf01c33d4a5a637d849c2b7a8b8eb13ee3e65965032990f41125391c94a5e7f63"
  },
  {
    "at": 1788585817606,
    "totalUsd": 65.410268,
    "holders": 497,
    "realizedCumUsd": 8500.040743613074,
    "txHash": "0x6013848ead75dcaa27641352eaf556bc65176a05d2b664f847fa6f4557bbeda5,0xac2d257c286a0ad95b1c17046707051e0bc3525d642f119f636c68e3127c0356,0x88fd43174f3ad3dfbcb93e958c7b1487e523b7ad2b18c1a3a57b735a06890e01"
  },
  {
    "at": 1788586713667,
    "totalUsd": 335.885611,
    "holders": 867,
    "realizedCumUsd": 8835.579150613074,
    "txHash": "0x3d5318b31416e8c081351d5b4e1b64af1d29d6b7cfc5a9c1945434c0d954f4a4,0xe78f864b86d836c05b1e67b58aac4859e6237aa785609d4070b582737faf0a33,0xa4c0e42e0e8f3c326fe8b4a5a50fdf10af5966c6519ffd297da9070f429f4cfb,0x182a6a114e3e5895509ca45bede429e6d061b8d3dedf32d1b304d3c4b261a995,0x832b93b58a9a6d1e2352602f693344e5d5fa47258a753738eb3b2cc2b25caa71"
  },
  {
    "at": 1788586762674,
    "totalUsd": 0.898223,
    "holders": 43,
    "realizedCumUsd": 8856.605356613074,
    "txHash": "0x394916d57055c11a623e03e04658e133fb911f73374b522aa7d04011f19d7113"
  },
  {
    "at": 1788587597870,
    "totalUsd": 17.097048,
    "holders": 280,
    "realizedCumUsd": 9028.659859613073,
    "txHash": "0x2995b2ecb734b154142828175757433cf5b0241a60dfca0ec2673d303bce01b8,0x0a9e16c88ec0c6cab9f5f16848c67a360f83d3ed18ff7e46e7f0051e23c3faf0"
  },
  {
    "at": 1788587648271,
    "totalUsd": 261.096487,
    "holders": 799,
    "realizedCumUsd": 9290.042110613073,
    "txHash": "0x970e09ab8b8644fa97f612e463c2623e2fef8533136c1436d34b31044acf70e5,0x0350d5afac79ab818872bd3207a56b955d50894f4b2eac6b47e58f1d57e8d04a,0x09beb6e519898b4a1340431d1ea581caec0209e18fb518cc184fdac9c4839075,0x91cf82514e39de0bbe115379482b9ac72ef86c6bb2154f1d9b0be785d9700cfc"
  },
  {
    "at": 1788587846827,
    "totalUsd": 190.292535,
    "holders": 724,
    "realizedCumUsd": 9521.973391613072,
    "txHash": "0xcc8b4cdfebda71ca1ad9b62a554d3749cc2a7b6dbd01b40fea92d44b244ff645,0x621800ebe0e82acda596f0bd9ebf58d6d64bbf2a442dbad37764048b92831649,0xafd4ba0481e501bde83bd4833c82a5f181d55f25130e201735c6f774a27128b2,0xb81c93a28666d4a00e73a578a8b812e2351f6e87a96642254b1980206f1ac65d"
  },
  {
    "at": 1788587908722,
    "totalUsd": 95.887661,
    "holders": 578,
    "realizedCumUsd": 9599.641433613071,
    "txHash": "0xb079daa5cf32ad46e9edec60a6446bef23527e45f75f2a059815c3516f663912,0x21d7fd3cd297a764c244b6924bc61390a6294b56c71c46e88c5b24a04ea1948c,0xfc4e2411c7fdd2ed6ba7d505cb1ec4613dcadc270ff454f0a69f7044b9da1ed2"
  },
  {
    "at": 1788588210568,
    "totalUsd": 100.031374,
    "holders": 589,
    "realizedCumUsd": 9725.399694613068,
    "txHash": "0xc6107bdab5c4dd0873585db652ff62cb0024186f8c811f28c323f04b01c45a61,0x7adda64b46e3de6c5055d458c3dbd895fe040ade1af103b85f244cdffb084129,0x106ee5b502632f24e06aa2b916d29b6c05940020cce81f9a4f7c41ca0de77643"
  },
  {
    "at": 1788588321410,
    "totalUsd": 79.860701,
    "holders": 540,
    "realizedCumUsd": 9756.335899613068,
    "txHash": "0xceb0c710dfc6a5856713292e798b87e5d804e2825b9286fc0b86f9d1d94cb3d3,0x11fe88cfcc5a87b2dd1c1d6beee66b6f75c76bd655ff161b437234fe25d8a08d,0xc429c61d6b42be8bd01fe87e73f8c81eba8ca20d3cab763616cea9ebc98199fe"
  },
  {
    "at": 1788588497788,
    "totalUsd": 94.76561,
    "holders": 582,
    "realizedCumUsd": 9851.05163061307,
    "txHash": "0x62aa9465c572c3443cf8eb6a2a7ae19c348498da339181fc18f904180655ee1d,0xda56debab0fe9caf9242ab59177072286c58992cb24e4afe9db035888dd3a099,0xcee6ffc53de11976d3bd96b5079d275fe34284c51a3b3cff3d1c2d5ed0268239"
  },
  {
    "at": 1788588661086,
    "totalUsd": 51.964808,
    "holders": 454,
    "realizedCumUsd": 9983.48362261307,
    "txHash": "0xfac6b7415098a032d632dc22e40b1620406031817ca6efb4c8b21cb6058726ee,0xff0d2ff533a9e135ac295239454c19f5a5730fb899f3d9390734d94ac97843c3,0xdebcb6653939cde6640d7d21a3ce14cc7f02b797bad904f7543238be6a7d7b66"
  },
  {
    "at": 1788588985251,
    "totalUsd": 517.48637,
    "holders": 964,
    "realizedCumUsd": 10420.054506613073,
    "txHash": "0x0320411364df1550806cdcfe556300db08e20f2e1945e6ff9928178e13e9657b,0x399af347a4bd53bc70f84b9b5f5d3d80633382128f67852c8983855cadd1ad32,0xd3c2bb0888a1d0ab7812a972a300c1629985adce5f5bd8570efbf4dd35771e97,0xe5dbcb31a07041c3f24c4b80400845949441223937a62031f62f7e8e14486f72,0x448f5ebeee0675a101e9cd9c9931d4ab244f7664d7a81b0e5598de5b1321be7b"
  },
  {
    "at": 1788589092522,
    "totalUsd": 195.269213,
    "holders": 731,
    "realizedCumUsd": 10654.768699613074,
    "txHash": "0x7c897a8063127195e34f12660a4fc4a0bd1319f325148d4c97a6933e056da76f,0x31b0816ae840483faa61d1d68f965a51591bff5b5f961ac4c89ab814dbc65f62,0x39d229f90ac668280341a6afcbe4a2021c9f87910868a263c192c06d09f309ac,0x704ced28724b3d27ae694deccddeb35790643425df74d8e4c17c34cfa9ccd5b0"
  },
  {
    "at": 1788589668365,
    "totalUsd": 514.783377,
    "holders": 961,
    "realizedCumUsd": 11132.091114613077,
    "txHash": "0x697e67d82e81b3aa2f99265760cdc597753813bf82fae6ed238ca0a0851b5372,0x17d85070a2cb1e29f245912e1d43c29cc82d0319a80517c081d361c9ea71b1f0,0x25c168723726d0b07b2e23d94d8a5185f360587bd63625d0eb7cbee5a75ff0f7,0x129ceeafe5bc604d99c076f8cabca545fbd1bbb1c176fb9edca5bdb61d2cb4aa,0x085dea013a13711a3c5e8383649b6712ab9e5c8eaf5af05d5cabab8af12ae4b4"
  },
  {
    "at": 1788589879154,
    "totalUsd": 134.792516,
    "holders": 646,
    "realizedCumUsd": 11269.085535613076,
    "txHash": "0x21cd3939f017a28613b1409831e76ba62e1516fc82742ea9d52841489e3b54bb,0x8067f969a9da908f76cec1bd17c2b849dddca0fdafe95cb5ed0277ba797e5d59,0xd43f5aa4cb5eed90aa2b23fc9efd50e06f3f7a9cb51f4aafc03a387550327bc1,0x76ca0457de2c580681bef57cde59c66cbe352808fdc48b41988c9159a9aa3dcd"
  },
  {
    "at": 1788589967362,
    "totalUsd": 93.394054,
    "holders": 580,
    "realizedCumUsd": 11358.571424613076,
    "txHash": "0x350880531b240f99f86c9a8b007b87dd5644b8076bd683dc6f1b6e12f056a809,0x23029664e363b411812ea6053c10d60f6bae394977afb3da28e499bfb05dafce,0xf0560f92e51a95308d6e19774672c8734b4eb92690068708ca009032367b2a35"
  },
  {
    "at": 1788590863242,
    "totalUsd": 286.371952,
    "holders": 829,
    "realizedCumUsd": 11647.256674613074,
    "txHash": "0x99c81e139fd335279a7ec2ca040765acb161b18720e23d63815e185eb1c016e2,0xaa08e8048bd9fd13662d1876db747d07d3f9c71000e48b85d8e6fb747bbff6d6,0x9ea8a1eba47612b69958ded96fc735513cc8b165fcf3fec63ffd8329414f2ce1,0x472a5f2bd6bf731c3afad3025572b086487a12e096f381f0fa9bf92c40889932,0x4285aba38891a54a22092e17847605d22a6af8fb7f8ae584b2c465633b8fd1c2"
  },
  {
    "at": 1788591200920,
    "totalUsd": 105.661803,
    "holders": 609,
    "realizedCumUsd": 11750.737442613075,
    "txHash": "0x26d16d500df8e3937e5ef4f93a18a3ae13b5dac3bd53f841ae1b09acfafec701,0x0a73b5172d8492a59c7731b2af139ce1b713b537e31f40f73af1e30269fa94c0,0xca610f9553b1992d6b1f6f227a96c94a15ed9e100f08a9d05ea338d4627cd8eb,0x5ac5a3392e2df80be6ac6c65552c3651d18029b877031ce8cae552e6ca5e5513"
  },
  {
    "at": 1788591454375,
    "totalUsd": 68.447177,
    "holders": 509,
    "realizedCumUsd": 11819.273120613076,
    "txHash": "0x0982d768677bc218b87a879b43d7377cb65aba82f9d36190bf258057f955c904,0xf95986d88bfbc114c473f7c8f7b82fd10722b5c0cd23d926824532b443998de7,0x2cd00b6eb56e30814c89130a97c8fa13d2b37e2c27a4d56dde23f95254d99455"
  },
  {
    "at": 1788591531348,
    "totalUsd": 39.986143,
    "holders": 414,
    "realizedCumUsd": 11859.088925613076,
    "txHash": "0x173ed11d0cdeea6d2124c3afeb01fbf2f111cd85a49cf81791ed42027ab80b41,0x6d0ac708a6d9a294867ea9ced11c83f0842fd0b1fe5bfd47b33c7d15ebda67d8,0x04fe57b16d7c03f9814d06c9e0eb43c0c666f232350f48631b2bda5034aa79ba"
  },
  {
    "at": 1788592480043,
    "totalUsd": 405.419341,
    "holders": 908,
    "realizedCumUsd": 12264.311918613083,
    "txHash": "0xb4e02380a3ccbf9f79823775ff924e569ef3ac169cff6cb0ca43a331730d9605,0x820c943650d684159ac2cd2fbbc1af33b57b12c0f9cf73d8fcfbc94decccf7cd,0xf774ec9ed42ed3faa915b687fc485a3a44da2ef4938f3cad0fe941775480f0d1,0xafb647dd1e0b4f99c71fff87c614794a8e7cd3bbe58d833e75cbd2d742aa16f2,0x6229166d8429a6d5acf9316a5ec859ede71ade7e1448d52abb4cedabaf43985f"
  },
  {
    "at": 1788594988265,
    "totalUsd": 325.89362,
    "holders": 858,
    "realizedCumUsd": 12590.231744613078,
    "txHash": "0xd4da8614b32a1752a66f0851256c7df87679a3232648faf7643ee2b4bab6c6ca,0x5423c7314c432b5cd0de6ea03c80b25b12994d4071495a76fff992f62ec6cd7c,0x3c4d7206c25a1181209e1ab333766c904b14cd36e70d8cbf411606e8291f14ef,0x7efcf48d5c70c5014ab918d8b5aec491e18e65bca5d03c7283e514fdda0d4e08,0xed6fe1413db65be3f9aafa4292f124100225ed3733848336e1a17b1c5bc0664a"
  },
  {
    "at": 1788596603957,
    "totalUsd": 303.217836,
    "holders": 837,
    "realizedCumUsd": 12893.436587613081,
    "txHash": "0xbbda4c02e0b2a6f64de173379ffbf29a0de7afb9a36926e243f9a90581569cef,0xca365db66e82814aadb4f81ee2be29f075aac0f92f2ba6dfc1d8a343869188fe,0x30ea0479e2effd6668fe0adc6e58913f3b8fcdb7b25e97b27ce8f2453158064e,0x33e271751d03a1bf381e3aad5515a1d1585989a6220a0fb1108651ff8af8606d,0x6afbc9eb9921ac60e856a8b19aeaefa629e867ca3e304292e3be6339596d83d1"
  },
  {
    "at": 1788597523213,
    "totalUsd": 1177.87658,
    "holders": 1110,
    "realizedCumUsd": 14070.741418680145,
    "txHash": "0xe4ada38472ca40b512aa6225548435519d9fc9aaf82efb1b29c1096c1917b30d,0x95944081fd1f0a6376a6ae2a782186ce36ccbd3465b839e4f36a24985cd9f97d,0xf6fa5110a5649fedfe55c0069fda2720d4a1c8745768157404bd08a743a48773,0x045746074c74d8cb7916c6b2579cd0405586f7b2944c2958fbd84ca52738a281,0x4065814d162fe3fc4baabd9b8d926db8cbf7f99e4e1650a7b01fc620b5ac8a26,0x7c9f12504c755354bbde6f959a2f707f84e2376bb6269d335be207b0c7e014e4"
  },
  {
    "at": 1788598437091,
    "totalUsd": 1136.302415,
    "holders": 1103,
    "realizedCumUsd": 15209.08463153242,
    "txHash": "0x863e7df5262982b60f2add6e287c8be8dd19f63ccda3c79580f4d2a0f20198bf,0x68dcbfbd1d2d6325b31aa0e7c1ce2eb51d71a1cd91de184f2d4a0b559da8bb9a,0x9d3b6560c99aac8bfc5ac0018696f40d3388a7f92862fff7f9ce14c4af22a670,0x8685defbeb2e608e1ed7b3d3b923cf404b90ea9c2638a6cec58be09c8a205125,0xcace489652b7b536d02885face2fdd3e98fe2e54847999bf1a2887e9b9ce7bfe,0x3c32a469c8b552821709f19fe699d242a0fc634954eedf0584e07bea8ee9fda2"
  },
  {
    "at": 1788599405910,
    "totalUsd": 541.981793,
    "holders": 945,
    "realizedCumUsd": 15750.303559349697,
    "txHash": "0xd56c4d11a236b0c6093aaad5a1ce1584a548e95c265e45d0add2650d5cd4eb04,0xcc88c299aaaa1a8194ebf24b470f4cdde8eca05905e96c1d35e53c518d29a5ec,0xcc8e273173023ca94ce1c84edc4b1cd3aba7e5b9d65a59e583995454a5ec6dfc,0x12e60b3280ca33ac73681842198857b27ce17606c8c58ca0eee3723f284eb404,0xd177eaae46879ef65e779e7f872ba45fcbbdfbda8ef3e3705a3c690ad92543b6"
  },
  {
    "at": 1788600328004,
    "totalUsd": 904.789718,
    "holders": 1055,
    "realizedCumUsd": 16708.99029724667,
    "txHash": "0x280e90bd5b5d73e31b8d1d9eff2d2efb3d621794a6c839922940be9f6cbef117,0x5ac8cfc45ec0c95912b8c831f47f5c8698511a22c9f077661b78229abac2964e,0xcd119a70624ebf875e7642cfce785cb4d56f11caa262db12723066e35de563d9,0x0a503c7450b1429484cca7b95738d89910037776da7cfb9bf5b43f2484da5bb0,0xdcf62c04b59379976d08394dca34eec3963abe4a20c37af2bad437db158d33d1,0xfe03976b66ab30c14892144f9ff26a2bd7a68fe0f5881258dd3c5d2a3e984947"
  },
  {
    "at": 1788601282205,
    "totalUsd": 388.182786,
    "holders": 890,
    "realizedCumUsd": 17042.515090628436,
    "txHash": "0x00a9ab8b4f6063b9f5fb9d2c0b3d34d64154fc34ef043664a1ff4f9fc07f7bf0,0x92a34f0772b9ad7e04252243373520a5143df723f1df31f3ff7971705687c6f1,0xdf2968537435e9245bbe659b406691d6827bd65477de7197ddb5fe5970cebe7c,0x2915f3de991ab0445e6b20649ec966f790c70c82abeec3fde0a377a2cf7bd42e,0xeaaa563e8a041470595598c22f087b75fcdc0516d08e1c65cc06a18ab2385a70"
  },
  {
    "at": 1788602242781,
    "totalUsd": 436.659001,
    "holders": 909,
    "realizedCumUsd": 17479.23553219504,
    "txHash": "0xd6706107a5234b8b2e68bb68ceec4112097bebbb476d6d9580c7b88a39580793,0xa21bf6c3a716343f606aa115972a982846240db062ae3bfc6222629a00fb01bc,0xad603c9cec3896425d01f3ca10f890fab9a197f13b4b239efdfee8ea2b2304d4,0x3d0bb862ded642d8d993dbb82cfcf9edd1f35ada76868e71b5e344a40d5aa005,0x0e889a781ae67502268430437ef03f31fdb3f53ca59648b7629bb1f10a27dbe5"
  },
  {
    "at": 1788603203236,
    "totalUsd": 349.156878,
    "holders": 884,
    "realizedCumUsd": 17830.451626376394,
    "txHash": "0xe7227f04326a46059816e8ed414900ae5a1c3aae4f50421e4e9ce1a427723846,0x09fedb727ab61387b9d40486aae0a90f2216351764a4f1d5e5a29f5595d4972e,0xcfaf92aaea66575c08d66cc4c140a204c6aab716ae68aed0342a5af4601c3d99,0xc0be34d168ba089d3579361149865acd896bc276797c611f6fab8e6b2bff05f1,0x1d7ed8f8427826c4d26015a1001d4838cfc720533b5d025257b012be9609ff53"
  },
  {
    "at": 1788604163252,
    "totalUsd": 765.300993,
    "holders": 1056,
    "realizedCumUsd": 18660.745055526364,
    "txHash": "0xfce77baff36bb578aecab9ee1a72ee781b62c31c67c8b9477ffd4ed6a8dfd1aa,0x05da2c46658ea9f6eae72c079e0120a72b37e51a9406ae823965bbb21ea749f4,0xf943b96673961c215b3f6902753335789f362f4dc94c1d76ecdc2c65af87f5cf,0x72df29082c1da964fa40f8e7bbb07f27210f549121f2ec80d6e8605a2ac81d31,0xb0e5c01a741517ba30190a1966241a592adde30cd0bd757fb7f57f1b1ddc0724,0x1354cc45ec942b61cc4186a19902f1fbcda27d0cfe285abfde5d028b5797784b"
  },
  {
    "at": 1788605123646,
    "totalUsd": 555.444822,
    "holders": 991,
    "realizedCumUsd": 19149.06171584478,
    "txHash": "0xf57e4082892f00c070fc3b3d0287aaafa92d0b3bc7bc4ae674eec899413b516b,0x08dff47a396a6865b25a4e9fb1237fbd0a6f4136157f41c91f4c63c244c9f27a,0x7dcbb36c4d4ae45df52756b6eef884e5701b51d09c6fde7eaadd1f8874c15b8d,0x87949fa015c5bcd6354813ff1fa4011db33bcd0149bab958881b22a4fb7ad9a2,0x6afcfe76e2caae4a5a5c200b82719dd48fd2caaa185718fe5644c9c4c3060829"
  },
  {
    "at": 1788606141471,
    "totalUsd": 400.405756,
    "holders": 913,
    "realizedCumUsd": 19549.58829149464,
    "txHash": "0x018d4be81af4d711839622065dc8f04b857fd24587914640941a86fa73088da1,0xed95d991ab3d3c2108e98042a1dcbe578df445fd03d58254392bc3bf8926ee17,0xaa7399ae710ce641f8c32741937538044d4e0d72231324f80a00676c7afd8ece,0x4bb8f962cb5731fcc061a22dab00fe40500433e0a3f34ad3f5227b823afac960,0x912e68e64a0c6bc63dc68e19e5d7243950647e6b3704a1036364f0a5ec1e454e"
  },
  {
    "at": 1788607101419,
    "totalUsd": 312.841219,
    "holders": 869,
    "realizedCumUsd": 19862.445767595258,
    "txHash": "0xb1e8c50d2178faf213e2e3ff8a7da46b1efcabfad91d452e0e89a7b5f331ad03,0x8e377a0a0adfc4f794251de5733b9fefdb430b526832a9938d259681daf0dd4d,0x2c0d19646a794f31e45086eff8ba507735bb1f2309b5d8c8d915dcf722d398cc,0xfb2075dff5f17ef55d48bd2d830ddd8dd57e5e3a9a5cda9b2a4def62d828791d,0x8afd18e4bd1348a7bc45ac878c86ae528888787d89686cf5b49edea5270360fc"
  },
  {
    "at": 1788608181819,
    "totalUsd": 349.529825,
    "holders": 894,
    "realizedCumUsd": 20211.894554281313,
    "txHash": "0xbb3a389e9d991d0b13c97e69281e9f929570ecd96c4b74ebbcc839c0e7aa32d7,0x8118567beb2fa6f70622105d6b817ffbb8cbc7582ffe834ec5f7faa9252764c9,0x93c4ff57118586852545e18d4ca5ea9aa647e1d4d3498274d4c3ea9af2336089,0xfec75a8c01d34ba867a2d4eca3060a2526cbf1e4c790cbb701cededdf6fd01c8,0x4fe1f17d77da0b4a048722e58a49cf8e473bb81dc4bf0195545ae2f668484693"
  },
  {
    "at": 1788609141932,
    "totalUsd": 532.767968,
    "holders": 981,
    "realizedCumUsd": 20744.587051197395,
    "txHash": "0xbdf92694916201f092382ae9677170191911b432dad7b6b7a53940d204c17c71,0x1361095354ede4893a51d979b0ebd21996f65f38e397a7fb442a754be9a3c92d,0x7c2205e9dc56268b6b2ef6c9c0dcca39f771e8af1bd858b5983b64ec79a56a6b,0x92a093ea69193de78a100647373be0881d6394b8466e9a525537ef5cca7924f4,0xed655b6d70fd7b37bee94dd331e2f21b2448c1ae8c56e1995801b1965d071c7f"
  },
  {
    "at": 1788610087694,
    "totalUsd": 687.678918,
    "holders": 1029,
    "realizedCumUsd": 21432.16455422819,
    "txHash": "0x040347c38d8ee2d71c8828960505a078e3f5cb2ee660158d87d75c01a98c17c6,0xb069f764040a459b5877e4200e66cab7f98eb2b9e0383dbbfb3b263ea2f22fa0,0x132851b215ae57293c8d6c5036d93da90d61f1256f62f6761ad7dc6515b72add,0x2571bfd28dd8481e3726f13a3dd821cc6f1141fb5e110d9227f9d5cb70478fc7,0x8f92ed481e8a7950a2408facd9adbe0880d77d4438cc1da1a4df1f2eebde06e6,0xa237c3fdd392a09839494e49b68975b3657a549db99e505e6a5717018af4629f"
  },
  {
    "at": 1788611048772,
    "totalUsd": 375.793029,
    "holders": 891,
    "realizedCumUsd": 21808.20483305081,
    "txHash": "0xd228807c394879a7af29ba413f091cb01dda1e2a62acd0beb9838dda646db0e3,0x71c6ed535dc29ac777c6ef5c35fe2a4aaf7f858cb4b013338f36d257259d88ec,0x6d1b99b8126ad44d564300f1b04223e94175a02be51997a98e6a4862e2d459e2,0x7a0ac338195b8f8e847576cd29fc429db7a9ea96aa07caaf8b7a0ea77030b898,0x5dbc935fe73b4de0ed31ebf2009429e674531aab24e5edc7bfda66825724cdb9"
  },
  {
    "at": 1788612006710,
    "totalUsd": 372.579969,
    "holders": 895,
    "realizedCumUsd": 22180.778020809696,
    "txHash": "0x9c6b341c50206d25b269786e465aa8fd4a05c74660c24b8bb86b9502eb99b423,0xf15e474d2d45b40949536dc634587f9cb5318b2fd0fde1e795ab578f99ffbb98,0x5ab8f158a81076a4c50b7f9b54ab98fe7d30927255004ce0b46ceaf196a45d07,0x8fd3b910cfd230b8dbbbaf597be7bf60ee4dd74f4fd2687a4e09a2bc62c3b9ba,0x3fd3c839ab6bf0de0fee64d779a32c75697881b4bbf9a0b511162d448b2c331b"
  },
  {
    "at": 1788612964573,
    "totalUsd": 540.453641,
    "holders": 976,
    "realizedCumUsd": 22721.110818174264,
    "txHash": "0xd848433c0ae7250f59cdf6e3fca509c0bfb32356fc4f3f42e81ff3de7a56e58e,0x6ecaa4e96993bf6428c40b33a9386e31930fb420d551ae9086c1a88a4709db04,0xe0eb803ebf531b0371bf107c0f6e1a674ebf7196fad278b570a543e44dcccabb,0xf1a92037a8ca5acdddf4d72c0b8af167be37503f370db8d362c9ee33f8017f8e,0xe1222345b9c8b9ed9cf7aba6c1b23639612c370170e16cf67f787d5556ed7a74"
  },
  {
    "at": 1788614167266,
    "totalUsd": 368.054394,
    "holders": 890,
    "realizedCumUsd": 23104.17924761362,
    "txHash": "0xbaa0b3541f7f4480acbee200ce10b4202b9ed36c758923dafbf559635c791bc7,0x4c9ba5cd8dca6773eeb60939971ee5fb1e85e0c2a4dd2c0898cb4e0d4e4f6bec,0xa32dae9193ff01c2cff84aa0fea5ffaae8cb76363839ff968b80a85e5175e329,0x42921cf1995b285c69db487dd57755eb18e5495762eeb556191f26532d0c81d3,0x2cf7655ded4993707f1bfb7c2d8888d091b26ac10f71ab23690fd0822a0558e1"
  },
  {
    "at": 1788615247114,
    "totalUsd": 614.697898,
    "holders": 1010,
    "realizedCumUsd": 23703.814246904196,
    "txHash": "0x097695c7b8d4fe0ef21e1dcf8dd176d2820e0366ba65e81e11f5b417c4969d29,0x12e8c7788f2ad3f581c4d1c27873e3acbb8cebe67539dcc685ae280481d69594,0xe1f0e1a83091d8f03389b6ca6fdfd80f64abe0cbe1d3076ad8fa0e87d4f9ddf1,0x860c13443a32589b30608b0c372d3888af1909742190422bc35bbd8219a652ea,0xac865a28fc5d002bc39941dd0528215a1a581b96c5b8a5b5d2e0ba718e37ea57,0x3d239e8c289c5dc75bdee7e0c28a468b9dedb795903ad61dfab727b72242262d"
  },
  {
    "at": 1788616207007,
    "totalUsd": 701.91777,
    "holders": 1067,
    "realizedCumUsd": 24405.673676790448,
    "txHash": "0x6a8b814a6092c6ce45da379fd20c78f9a329748332479a960ba7a22f7eacc728,0x014c32ab0cfaad4588493f592ea71397e90b55d532f3b178c332ded9fba45b4d,0x58d139322ad9ab0bfb7d920c944a31707c93f7fb3bfa642004c744a500d11c40,0xd870aff4a2c2e777d02f4b98387a1a18b971522064564533f27a9d1f93e1ef43,0xbe3390aa23dafd3cb97ff53e2d35d9b0167ebc57739c2802c06f3eeafb7bd0b0,0x3dc3236346c7a0bdba5a2ebb83d8a044347fbe29ee62eeee609dcdff1fd3e84a"
  },
  {
    "at": 1788617166653,
    "totalUsd": 1514.414685,
    "holders": 1283,
    "realizedCumUsd": 25919.77454551742,
    "txHash": "0xfb07d1583312058f9469cde4d757b4aa0898729d6db657bd339181bc193cd581,0xe3d6bc5a343f6f1d8df7c0135a32b44f0ee9681a6953ad02705b83e87b1dc815,0xe010f513110fda5bc7ffa697e4f1ae79ba4e2edbaecc70e9cb9112b26482a690,0x10a028e6f6e75f042a4e0d8247e5503fb61d7510ac507324c3db50d0ed084851,0x1f5195781068395a98be5a63a5ae2726e4540b2b09f26ea54d67bdea85748bcb,0x8c02b9cbf232d471e65d3f714e4dc3b3e0e6e987c72f28248d0d023a15fe7351,0x31c7847f434cec5c3e34b247950317ff7d3f91a1f2fd8da973896c480e3f79ed"
  },
  {
    "at": 1788618130616,
    "totalUsd": 1096.401638,
    "holders": 1227,
    "realizedCumUsd": 27089.692267531613,
    "txHash": "0xe36c4d2a7282ba0c405e3edb6dfef389041b9fca5a8ac3ca61799d1cc2df05e6,0x5a3551e5fbedf9208ab1b50765a95d42a29863ecbe3e7ee6556890a621dea65d,0xa84b1fa68c576f3d2cd2b8959638eabc2972b0c317dcba73662a1bdd094013e6,0x8a3604f16cbef38c52a36cd8229dcd9029b0b8e2ab0c48962af6b380fa810d7f,0x9aac9b6b7b5e35d2b8e47ff1f776a1ec358c7a892dee5c1c522564736fff9fb5,0x7b566d5a896fb612b76419f42f385ae13ef7ed636cc2820e25fa3899a056abce,0x802f3abdddec71b779056d15378847917d760f9419a74ae0df06e9a654f1ce97"
  },
  {
    "at": 1788619088438,
    "totalUsd": 622.093033,
    "holders": 1093,
    "realizedCumUsd": 27638.884896571013,
    "txHash": "0x68e6a06edbb8038cf5bda78a195744eea63d2eef74a9fc36ef73041eb53972fe,0xe2fbf6f647c2ab1d042cfcc19d5f7ab552dd6cf2dea9346dca774981ddd3b5a8,0xf973de6505bbafdaa0b7462a9692df1e75f3abdcd31f298779e0f30911e343fb,0x46597d14615a63d09ddf3b51f8d1b6762402fd0a2bc8b65d12373db7a7e42e6c,0x3fe00a03d77646c31d3e105170cd7fd164dd192f94444456913f93010150d245,0x673796d940d0d037974d585d5ee19139c7882eab092081fe372ca6bda31da14a"
  },
  {
    "at": 1788620108170,
    "totalUsd": 591.794887,
    "holders": 1080,
    "realizedCumUsd": 28230.666709040997,
    "txHash": "0x7ebc4c181c553398fbe209fa6357c0f9b7a7a9113fcbccab0559b30e0686456f,0x6348986dad58c390d3a89547a254695ac3a1b9a05cee079060438684178f74eb,0x066707555cf30886220cb6eeb407afef59e71ae714f7397dd991df93da5b6133,0xdfbd8f8be2a65bbae10230e5c4b12b705bca8e891818c9c292cbc67c7a177ca7,0xa4d2fbeb47f1bbc7b63bc3b558aaec59ea19724b73aa52052d4c3fb19840a1ee,0xf1cd82e62d295c8dc0e9a92a2d32da800254dddaf26fbbe447d4faa24597815b"
  },
  {
    "at": 1788621064172,
    "totalUsd": 555.7312,
    "holders": 1063,
    "realizedCumUsd": 28786.364979031856,
    "txHash": "0x5ae8e2b96cbca1fea15bae8e334c2c9f8e642c07a464d84c775b4560ac3c2054,0x8232cd58ecae378658813d3cc808c8477870d5d068fbb17cf0688e52f53f0ef8,0xfe7738ba4fdc594eab33b9dcfd4a8c28e78ad3becd457ca64c376243aba7f13c,0xe25b805ee9ac75c4924f9af681f171e823eb9eea72643c9f4d55f9f064ebb4b5,0x47e9ceb88549d8bd9c27c6efb895d8b6cec42f934160a65dc1444b7ac4295a0a,0xdfe66f54d171548b78e90099f266dbdfcd59bde14485218cc8e1f263181a541b"
  },
  {
    "at": 1788622023188,
    "totalUsd": 653.167879,
    "holders": 1087,
    "realizedCumUsd": 29439.51241045031,
    "txHash": "0xae0ff488a381a5c9307b26ad711116b9529258b3a1a1448fc75c142cea3403ff,0xd044ffcd890833404f94030b320d6ca7a3c8d379a7c8e9be6d0e627c9f9b16ee,0x9eae60098e20b11593cfab8c1ded02b0f71f38eebc8c628a51e4efe53bb95951,0x262eae7b6edee567cba324832d021b082ac5bd3aaa94015123556c9faca73e51,0x6660e2690a56a58acd0336105961979d4bf5254aea76465aea75f8d611e8a0a2,0x108eff5e5e69ebb5e257611102130b1fedd5c714ad49da19eb62f3d164119af6"
  },
  {
    "at": 1788622984011,
    "totalUsd": 509.772737,
    "holders": 1026,
    "realizedCumUsd": 30005.44154184474,
    "txHash": "0xa48a422aabea3fbbb612a189a6bd6c7f99899c5c7f6a600faa0520037c2f7362,0x04d6bad60b183d27ae7bf900909a15c309b82baa434b9d2805133dbaf80ce245,0x2f384085b2ae2c3f632c133f1535329c2e6e95d9aec100642376e4f4975b9961,0x983a99184f20451d15557d761615c72c79ea6ca9b21dc05ef75859f0cee2bbfd,0x0c9f7ac4bfa8726cedeceafab274a57d5f17eb733224850c08ccc77e88d605f8,0x0bd2b27d0ed5cba0d52d0229aa79e30755589fb54badf8dbe78456e14d98076d"
  },
  {
    "at": 1788623941169,
    "totalUsd": 500.068294,
    "holders": 1024,
    "realizedCumUsd": 30449.44236927682,
    "txHash": "0x8c1d3bdbdf9fb89cb43a3cee8d04d439a1de0a08b88bc14fe4a56df1183301fa,0xf2bbbfed5ab5aa9cc88dfe4bccd7ac11128d2a6263e90648eaba5824d630ef68,0x7e6806c47231f5a41d675df4f76316e7a7116a64f6fc1b795b7a2481efa912c8,0x8b92fd259931fb377943757a8f593511d422e5edf768202b0c3c8c2f8a25ddfd,0xa439c7513008776fa8ae9d9fa30401dabe7ca1230009778933d32a00c7e9921f,0x6ea5e043e4e1ee638989ed9e88bed2e11ee5e536eb7d417ab0397ac5c3c91f1c"
  },
  {
    "at": 1788624901622,
    "totalUsd": 1376.766018,
    "holders": 1263,
    "realizedCumUsd": 31825.63906711978,
    "txHash": "0x759f7a9a88c8c59f1aa00a2243f065e9853d6566fe0427fecef7b7f920d073d7,0xd1b20a078d6016612d06d9a4e04d5b0567f4afc440e82f5df62d09fa5e522c41,0x711c9fd974892b05d86cb631da35ab5a48ee762d5b63d5cb6d7bc42d24cf95e3,0xca7a12bb52e395705b6e7cac640b53c8ca3a9be98e783680c894388790659fbd,0xea9143c090782f7d0db8c5a31d05a32791bb9ba1c10d0b4f180f9436f738da10,0x76a55692c9fad444127cd17a1cfff73150195e2d705c6db5b60ecfeb4605424b,0x096e5c87f8a911889ef3cf88798c9ceb29613ff13906810aea67f3c86866fe69"
  },
  {
    "at": 1788625870906,
    "totalUsd": 419.978001,
    "holders": 984,
    "realizedCumUsd": 32246.222638180858,
    "txHash": "0x22fd249a4abc42132e8b917f3952d5fbbfe3a448298558b2cdb849769140376c,0xaafa2c6fc6faa4cf3510c98f9a907857ea025c2ac525f4de93f01f78f2f20eda,0x4edb1e7074a8b5de9eaaea9a3dec85509cdc3fcd194fd93b4c790890f0e5793b,0xab499dbec1d6431116a7f28eac10ae02ba605ed0e4d2f6dd84bf4c5ac18264fe,0xf45e0c4866bf9335a2628cbe1397fdc14102cab692ec5808b8c63d7517f849bf"
  },
  {
    "at": 1788627002193,
    "totalUsd": 405.691593,
    "holders": 968,
    "realizedCumUsd": 32651.956916091047,
    "txHash": "0x6ddbe65a384f0bbca8b8fd65a71b0011129839de5c2b306f8e4dace8be88924c,0xbee3b560409a0a71af6e7d4304ea0af50bdab068a5b3d3bfa82f88d5c91cd85a,0x80f527a5512235fb3356c4feba3ba31295349ea69699306a1e488e03dfbd83b1,0xe45ccfe3280c9a0708dac6e05da870103939457178dabd0398020edf745b325d,0x8a138e36a82f3d1fd96843db1bc59465fa90175afa957221b14431a59b52e8b0"
  },
  {
    "at": 1788628201976,
    "totalUsd": 407.606641,
    "holders": 965,
    "realizedCumUsd": 33059.58175133411,
    "txHash": "0x2c0284c10279576ce9dbe5f929945d6faa7f6c4b6f6551634969685a92c0ae0a,0x7d785e8039e2bde36939dfec7e2b7a642f132d2f09952bf170efdc7cfd5bbcd6,0x3a794dabc742fef9254d46ef2b367cbc51b30502429950075d8f5debf519c45e,0x53a685f2cf0db342b27b0b2bd6f67b32259817e375bdf91276850d4530997042,0x84e7554eea7b12e3378f7862c66a0bf249e97291b2be006b7f708f3570be970c"
  },
  {
    "at": 1788629160839,
    "totalUsd": 407.680932,
    "holders": 978,
    "realizedCumUsd": 33467.273549806865,
    "txHash": "0xf20b1e55ffc154c3d33406d1f33749bed698644f2851be909b9a831176769197,0x6f2dd494fea0cab7d8bb972315091245199903038f5c2950e70d81485cab30f4,0x8370b3ff2b3c6c431b03d6c0cb9530f4ad71f37a14f57804cbeeadf8485907e2,0xc34bc76fe8bfbdd75cb6a7930ef70a4190a9c40feebb0a163153963a054e26fa,0x004d6a192012f38bde2034313a27387819abe31fb8e64812f9839bc0916545e4"
  },
  {
    "at": 1788630121223,
    "totalUsd": 303.005625,
    "holders": 905,
    "realizedCumUsd": 33770.36892780686,
    "txHash": "0xa4160e71547383a38fb9c9d063b5dec74db38138ec0300ce64a80ba1e3714b52,0x29a00b82cc169a5d38d5d122475511fac52a22f65d4102817061113b3092e829,0xf171e96386750dbca9dcd815a12f903899359b34d2342402e49a119a9b25481e,0xedc2f1c23abafff9bfd96dd276071b683afcbb8312f2f044b769ba1150ee315b,0xb30b1860cb44b5a7cf24593b0eff3b163920778dfcc571be69212bbec7bf3582"
  },
  {
    "at": 1788631087147,
    "totalUsd": 413.456449,
    "holders": 973,
    "realizedCumUsd": 34183.730603149335,
    "txHash": "0xd539b32a6be237363bfad3d2c32f0b8712b067ce5156a78fbb12cce57e039aba,0xfd7e4309894cd28213303e7d77ffe0f3d6b4d65e312f2414bf86255f680ff1d2,0xa56eda5841e0c4980f6ef98206d844c1da2433b0b38b170db600a84cfb1ff419,0x92daf612328363aede156b2a5fa1001b35d95660e806a665d6eab892b5a8a16f,0x747200344f3dbaeb0e3a0908f8caa1f7cc0a42c39f7386905b011aa8fc1026ac"
  },
  {
    "at": 1788632103282,
    "totalUsd": 395.199973,
    "holders": 960,
    "realizedCumUsd": 34578.90085814933,
    "txHash": "0x7707cdc1dabffe264c3d587c9e3a180ab14114ae58172e20c951b6d66a178a31,0xeb2328c3ab6286af96e3cb8998697017a42ba2852a311a6778fe8d084eab7e70,0x05cc0685244c0e676f8eee8a60d8a022a10c6b6c50b09180cd31ce8dbd5f2be9,0xec8f25572d63cb2143f12e4b0d91e589e8906b74bc0b150deafa340e042657fb,0xdb08df1032b125074232e68b2b6eeed9fc6941275d6c344639ca4eaff37e1161"
  },
  {
    "at": 1788633060515,
    "totalUsd": 409.452175,
    "holders": 972,
    "realizedCumUsd": 34988.36177013592,
    "txHash": "0x15f5d3c12a4de40454a82ef34a1a9ca735a5d5836dd8e730483691ed190880e3,0xa55b7d89070feb722468d0af489e6673dc367519f9e24c57b660d62315fad0bb,0x7bb98bfb423acf30c32edfdb51b99043c9632b31d583ab9f6f871b3500df3eb0,0xf71fa42641931bdc991de41a68b30294900a042eaae1736b295b6dc4f967d8f3,0xa3cf879a8e48bfeb014254b053694c29286f1eb62c87e78e36d81f9bc92324a9"
  },
  {
    "at": 1788634056083,
    "totalUsd": 323.638323,
    "holders": 918,
    "realizedCumUsd": 35311.99374185648,
    "txHash": "0xef6097e4d468e712b09b6a58e047f5542c37348ee72802622d7a99256fc55735,0xc29d84fdd798a0c7ec2d55735856ebf1a3fdf2ee03d6f2aea30442fd6c331b96,0xad2e397d0ac65c50697af3c49e689de8bfdd1569d60a4008faa47334d40eca1b,0x7ed0e9c829e468551f3c61b50d6840d7613ee09d0e82f40d6365b2b1e2700c26,0x7ed99fa4093da380afe6b9178cd20703688b10863880ec43ae7a03e1f1886faf"
  },
  {
    "at": 1788634979934,
    "totalUsd": 1223.165603,
    "holders": 1221,
    "realizedCumUsd": 36534.53455104378,
    "txHash": "0xdb216a5a4fa1e515e6c5db22dc8fdf125fe3d164b34c061b6cf280bebbceaadb,0x53782bb1d0a88c6b25ac41b04ac2e53119ff49865f3fd2c1677c30a6808861b9,0x2b3ad9c881780da05efd86c6c204aa9bb3b488d8637cf5f8497101de7ca8033f,0x14ee76add09319574cebd439a90e1a9b5af0b162dfbddfd52b27194c34ea8416,0x813f948eccf6df555aa79e2319e0b45ff15787f027a84c049f57b7ad7a61931a,0x4ba98718acb2f575d8dbdb020e43859e1e3774f4b274d422235e48687d10ba45,0x2b6447b2c7b8214c51c854c9844138c48522bbc9857884c4c0d772ebb25760d4"
  },
  {
    "at": 1788635937855,
    "totalUsd": 545.35497,
    "holders": 1033,
    "realizedCumUsd": 37080.31892800102,
    "txHash": "0x383ee96124505c3d3b58f38cf6a8ad929996af5daa54381e3689b54fb505d584,0x10c7b140e8bd2dc782640a04585265601bbc85029c596d5a99339c8995289416,0x2299bb714f2e088297e0dbc55e608cdf7370e2093763a914da4373c1fa47b4b6,0xfcb5a6cd98b3fff1f3669a059449f2ec7c722ebe41abbacc0fcc7bb5127a262e,0xfde5d4fae2178dc3dd723f480c0963cd554ab68900b03a7d27eb7e7558da14b3,0x95bc5779bb6d53dc55d741f8c0b0b1c062af20d16a57ced4b57551608af27eda"
  },
  {
    "at": 1788636897167,
    "totalUsd": 1330.604328,
    "holders": 1228,
    "realizedCumUsd": 38410.41017526764,
    "txHash": "0x8def3665d1f691adb7e83b7c40ba8128f841f1a3ce08fa6441a425c2293147cb,0x69a3bec10c81dc924913102084d6e0dc902fbd91d67fb92ee8f3c726ac85d0e8,0x8b585d4c6a53e4a01cd5aa8bdbfe2276c406a506bace2300dc4681a84ed4b851,0x86d4353c87c5e6e5eaed8e49d8b7a887873e7851e5930fe88e7fc0761da763ff,0x72811353deed6d135cad1362828901533972cb28e5cfb5a5b5524b9ee1c908e1,0x6da08ccc62971e2d6564c6181b7510450fd84154729f6cb69ea735a003fe65c2,0x14672250eb0473bf17751b0e075a5944d136484dceaef8134b1bf9461bb0c9e2"
  },
  {
    "at": 1788637858984,
    "totalUsd": 653.385513,
    "holders": 1073,
    "realizedCumUsd": 39064.197174562534,
    "txHash": "0xdea22e51ded531339a15120f8db51aae2f4cbf867637ad6566dbd9786bd6d5bb,0xb3fd94cbcfc4f05fe3190f4a5c544ca5913c015f777341909be661e3568b7203,0xa18bbe41403a1ba78d9877670f1119d6c3fcb62506a5572887c4cacd96951814,0x7e3fe57f6c449f91568166e4c091c3b6a543f6c52cac012aeeb032b5d18979a6,0x08bd7a6838ea4aa2d6b0eb1344e0f552c1fbca87d07eb48382684188a1f54549,0x9d7cf5123cf90a2f04ca9ad8afe5ba467d23baddfc3ebaa06baf98b5104f2bb5"
  },
  {
    "at": 1788638833057,
    "totalUsd": 917.326736,
    "holders": 1145,
    "realizedCumUsd": 40044.321761767664,
    "txHash": "0xbad76ce7fbeb7ab75d7cd840a2e82e082c8f46e60b0136bf72ecc852340c8bc4,0x0e77afcd992122156f2d23a9ee3a1640f3b69f3ac963c2092c67dd12631e0e25,0xc3faf3eb0c56d080b678252268fe5b192a3ecf044f10d1a6336a49d9451c37b5,0x1ec4f1ac15c3e5b5a5a6ef6654d04749f2e6d46c3169fc092077ee064f2b2752,0x5d02acf968d04b01d1bb2d42ebc68b6776e5c6fdc31d716c9c1654a69e7a76d4,0x5d12e39c05ca2cbf370988d6183743fd60868b287cf0086bb70d4ae5aa8023e4"
  },
  {
    "at": 1788639808190,
    "totalUsd": 948.563029,
    "holders": 1146,
    "realizedCumUsd": 40929.95878261052,
    "txHash": "0x5b5a59eddf081ae865194f60609451e81d754791fc1c9c3cb70c8eaf767b9900,0x1bbbfb20fe1aa01b2e90674236a94a51158c9b61debb858475f0d52b2d92e213,0xf41b4730e47cca12a4addc9566107965d591127650b391aa6dd8e435991eb711,0x4db9925dc2ecd8d93633dfb47b2bc53d8d774c2d0541d959be07ad8bbfff8534,0xba5926f1939f5dd64fbc7ff2afe16222e0caa7493cd997aa43bbe80fbe326474,0xf69745ccc88c56f0634b7edda5a41041be5674f46a2e738f7b495bc85c898968"
  },
  {
    "at": 1788640745728,
    "totalUsd": 2779.558601,
    "holders": 1346,
    "realizedCumUsd": 43709.00954573475,
    "txHash": "0xcf0fae3a7d9ede1f56c4e249652504692f88481bab90ff76d92a3b871f1d18b9,0x9df07f37736201e4b9279781eb45b0e6c05f9770adb5b1039d8ed549cf95ad47,0x7d839fdb6e2c5c1dbafaad7437bb4521d5b97f49f98f98d1b9f47c008e89bbd3,0x671cf18d50b8850857cd2c99f226841cf23fb60eac0dd96196ee666468c64ec4,0xf576ce66d003711287381601f62aeec649a6555fe7d628290e5e498193c2117a,0xab79efa579b4319f953304e696d3e27b1febfad5300904ad358182f6a754910e,0xf954c0cb389692b3cfef66ecea222bb1d60127a8c26062a9f76eeb70bf3af894"
  },
  {
    "at": 1788641680695,
    "totalUsd": 1343.880964,
    "holders": 1232,
    "realizedCumUsd": 45053.12500818151,
    "txHash": "0xec2d8d44bb2a0ddbec9fe76c538728b020134772bde1c833e0f92460627e7316,0x3c55ef3bfb8681ab58a68c524a655144467656dcc6c63afeebe1e07235b94ff7,0x672ab7949b4c871497e3165dee6590505f466c9a43aded1e9536a84ee9fe5b5c,0xc9b191ca6028c002decc7072fff93f28013315cb8c1614b091f9ab9bc7387ddc,0xa7d0151dd88e77570ed59eb5c6ed823d88ac5330ae7674e4a3fd080c51c468ad,0x830531a6da155e1ccc9811b47d77fd0db80a8863c4b02335dbae6684363c26b4,0xcdd34223d0138509e38561196d2d6bed5119029049aa0b8584ef8bb159d9fc73"
  },
  {
    "at": 1788642633379,
    "totalUsd": 366.695739,
    "holders": 921,
    "realizedCumUsd": 45420.47606310485,
    "txHash": "0x5bfec9578d045e7b826c97a689eb2515e098df0847c186189a9c677a0b6c3a46,0x17bff6886707d3330054eb6293dcd0c420221c7cc9da39d61240414a58176406,0xa797832a2ca0564c46c46eb804b8a1d3752deb8b084ca7aec133e6bcc884e2b2,0xc1fa0e7c923952864d09e3fe226674905ded2e86fd0019d4ef348008481a2653,0x8ac8db1ff1d0df13205c6247ae8dafde74b1d40bb7dcc32f48993c5e68026799"
  },
  {
    "at": 1788643583306,
    "totalUsd": 333.708777,
    "holders": 895,
    "realizedCumUsd": 45754.20699403508,
    "txHash": "0xe33764229eea220403cb4ef69f5a9498033143f76ecb17df0ead6113257837b7,0x03f3d9474fdb0d756aa86f503f3716c91bc1b0b8c99b35f004c434fd56dcca77,0x3500aacdaa1158f5e9d589e50459c82505d3b37abba50022cc44515ce33968e3,0x3bd633e5cf90c9224d05eb9964033a8f37c0a449335ac1e047e85cadb1bd41ee,0x8b0263de7b1735f1b03eb36c74d34c8edab8b118f8da1ef2922d3ebb0609a367"
  },
  {
    "at": 1788644520216,
    "totalUsd": 333.88877,
    "holders": 890,
    "realizedCumUsd": 46088.104520832385,
    "txHash": "0x5e108458a68f2985cc069cbfcb975fc8d1ae1c86be597f9f43b58f0eef1f0365,0xc16903bb9b91d9cea0551749b11605249bb97d38236dde22d5969dd5808f10ad,0xc90d99b84e3f95e2e94b0cc75ecf5b5502060c7a9be726437b0f48aaca8b8fb9,0xc688d2f28cbcf515e852a5782dd7fcbe29c844d55297dc5c22a9ed525ec8ebe3,0x135ba4457fa82d3f0930e3c0696d0cef35f74c040808c9e34ee7ca4e7b750297"
  },
  {
    "at": 1788645463524,
    "totalUsd": 624.375051,
    "holders": 1033,
    "realizedCumUsd": 46712.25628311848,
    "txHash": "0xd57452f55ff69137850b1defddcec0a35044438ce5e17163465ea592740fdcb9,0x7693ce6281a6063fa6c6f82ba2289b525b2e2e1a5feae9109208a73c172a8f31,0xac636026323615ed24d87ed9d8b86a420128d4042ad88add62c96de0739dc417,0x5f02967db374db8feac2d2fe3f2c699871f70fa3d83666103455a66089b255ba,0x13a2bb77b94afc09ee37fb832d8cae01611eec9c2df86853382f72eaa39fff60,0x8581c0137c5cabe071f950e91d7c716dc82380509ffa880688fcf27fadf6c8da"
  },
  {
    "at": 1788646423264,
    "totalUsd": 462.812882,
    "holders": 956,
    "realizedCumUsd": 47175.14257698002,
    "txHash": "0xdfe9b597c92adb3f5fdb61f6b1908d2b1fd2931174b628a7c8dc883198362fd4,0xda084057dbc5fb3fb3bb73380fda5e2b970c5f71bf0a16cdeafb3100d58bd89a,0xb8d007649929a86317cf6c4473042876a1a780902c995a808e3f9fb170e7f818,0x24f38db2f43cab29354031f7b3e560576639289e1753bf28abf8ec6c1eabac4a,0x98a5fa11990f09f0c0129535d6b4fd1ff9b91b757cb91bbdc88f96badbca5b85"
  },
  {
    "at": 1788647427621,
    "totalUsd": 304.538093,
    "holders": 854,
    "realizedCumUsd": 47480.28497044384,
    "txHash": "0x0746fad7192eb642923282c3f0db7ceb01651c744b42e00b4f609183dd98a633,0xdd1f1a6cdb10df0ae268eaae9c9d020d161f47bbf18d2ad01ff32156056cec09,0x34ed51794c0a517369a9202a6da957845384c0b4f83425bdfbf38a41927071b0,0x73560cc982f007f799e918ce803aae2795f6153c297cb89c1fbc6d6fe251c096,0x6babefc56450e6b860ee9f2b40ed60637660e9c165e433060cf841757f137553"
  },
  {
    "at": 1788648372939,
    "totalUsd": 628.798393,
    "holders": 1013,
    "realizedCumUsd": 48108.349245756945,
    "txHash": "0xd2ff5abaa86ef2261f7359b172370ac16a0346531feea4561f89e5ccb07b73f5,0xe615c36a93cf8c1d8c38583dc5b585f2a3f49076ac562afc22b92b4ee3923131,0x7cb0ed8b947eba23d7ae9a9f114e853ed8291d1817c18a4bfd1781ada5a79a96,0x9eff11c12e4c36d3cec212874aa9f0582f393c5e5ef036e1455053e4c6cab6cc,0xf50ce0eb127b7a39fd5652a671f6170fb15f2bcc63fa9eafd33ee4271b983069,0x0feeb2dbfd2787c56d29cdd08cd23dfe9f4c1e4009ce3bc46763167555e0d2af"
  },
  {
    "at": 1788649332671,
    "totalUsd": 702.25851,
    "holders": 1041,
    "realizedCumUsd": 48810.60919808215,
    "txHash": "0x771125b47781c00a0ee2a47116c7f98abea7ac055f4983e9a13cd74b0eabec57,0x25cc4956afa06b9fe756840647b65da0de65180ef6a4dea045b32e17a2ae8f1c,0x5d25efd5aee6733fc5044f5dc77a1a0d51a63393a61b59135ccd2d4d2c9dc27f,0xe80d8502a0548c7bf63774dc0d596ecff454d3711a974f61085b87d62aebf91a,0xda6b5516eeb5381aafd24c7dc2c0b771bd0a4b0e3ba6c4a92bbab05bb9995b5a,0xada08da449d16ac1f1d5077dad62a5c5235d3469e94ddb856acde6948ccd2cd4"
  },
  {
    "at": 1788650290806,
    "totalUsd": 305.945811,
    "holders": 845,
    "realizedCumUsd": 49116.8582435298,
    "txHash": "0xadbccafb988054f82268aaf16f589c31dee08a8563054c3286336acd2279d01e,0x2e2d2b89db0ccfc2f90d80da50f4cca2461da6de26454b5735046fff16639ca3,0xdfc0d55ef28b8b12347df5b5b833b8859a97ead9b87c80e7ece56355154c11b3,0x39908a7f78ce256370daee217364dff24ac76ed0501b383cf36a0c2045953f67,0x56abf3bbf5f27d548e1b14614cbf1245b94478bf0b01306be8ee645d2bf4ec83"
  },
  {
    "at": 1788651783381,
    "totalUsd": 303.766555,
    "holders": 835,
    "realizedCumUsd": 49420.58522790269,
    "txHash": "0xd205b9cdf1f4d22ad382aa572a3ff2d72c767e12053436256519f88bc4c081ab,0xd65fbc7684e48d2517b69f78116ba426081176c78316acef1e73c26fd40eb471,0xb44b91d9729629ad099297467b808793f9400f1fa94be44cee6f605018f24562,0x536ab435cb0524bfdbe3572e314710c79dcf785eaeea3f431a08bbf2c75e87bc,0x63bac4ed9f7d0a412d4ebfd082705e851d44240a428db5d728c7b5a8d6c26f47"
  },
  {
    "at": 1788652743978,
    "totalUsd": 559.951258,
    "holders": 949,
    "realizedCumUsd": 49980.34464315575,
    "txHash": "0x525c773dca5a1b7bad77452dafb9dbde7100cc7659beac993f63bbc68bcdbbaa,0xefde3edf44f163a7fea7874f98715716fe76f275ad8433ae836290b3eed2a606,0x64b7ebfad6a6dd9bc48bf4f3e9c6ffe505a08f2063749e4cd70730ca2bd394c5,0xbaf23e9e3c90b55614b2eceeabd9c853bc03e92cc92eb684789f0aa050977380,0x7c79bc5d50e7314e7636c5ca103ad7da52611e15a1107a1dc2d990a6b1174955"
  },
  {
    "at": 1788653688220,
    "totalUsd": 526.37366,
    "holders": 940,
    "realizedCumUsd": 50506.6610958001,
    "txHash": "0xc0a0320d8835c6395bec01befdaf64e6c092009d56de545eb5f01669d20b489e,0x445abe3bbe212db677d9f84f617966fc9e39a1966d1b9b68db342cad1d852d8d,0x8680ef7f35251c6a73553e9611ee48ffcc74d48e4857d506ec0f3b7b21259ebf,0xdc0af3f1ab43963e59a21c7f166dd24a21dce2cd0aa2541a3abc2b00b1e75287,0x5d7c7469984ca5a509e65a6655f76c4b2f6c0c10fb87a3d57de0fb53d9bf5ec7"
  },
  {
    "at": 1788654634862,
    "totalUsd": 695.882737,
    "holders": 985,
    "realizedCumUsd": 51202.475178284614,
    "txHash": "0x8f0fe7f5d89f4b5f50e59bdbec715b15710f08a08a3fe2e1d5f546040564723a,0x9c09a239c62e04e8a439e21c13c9540f14c7bb956cd45c39ed9272dfa8d54732,0xa993da846a96c12c3a7d41bf20624ba2389221685994a93c0044d7eaa7dd16c3,0xe331e447df328da821b06a5dcfea723bcbfffbabb94c4499dd7cd51101095f85,0x697a065605342ccfcf7e7bed6c050a133c800aa3dd310907710bb56f1e38c446"
  },
  {
    "at": 1788655595470,
    "totalUsd": 644.401211,
    "holders": 961,
    "realizedCumUsd": 51846.850767832744,
    "txHash": "0xabe1a92096df122cf63ef4e281137a16830cce243c47117bf771a0a08f6db88a,0x40c48767195e9d88eb707dc9e30839951c5ba733e68198681b331c54cf80e110,0xb650d8f567ffd7120a5d06a9f0350633337387e9c7d98cb8153a3a506739d225,0x0eabce1a392d5341ad9a0b567bb5ec693466a505ec00b56a23e9fe53d90416d9,0x8a9bf5226a780a1c3010f11f4a9cbaa377468cc1c1c74440879f9ba244a2ffda"
  },
  {
    "at": 1788656611418,
    "totalUsd": 531.283725,
    "holders": 928,
    "realizedCumUsd": 52378.15416114238,
    "txHash": "0x642b58032e6f947e3b353c40fb5c2391bcfbc40899776847ea315b8652ebfafc,0x76f640e9c9785bd916abeb79573176d693fcd003c72817921969c373360ea9ce,0x249cc9497e3ae02f57c79fe6f80a8e983a5858f176381e0447c06a96680d49e3,0xccf5c4d02e12f91a1355c045e94af0cdf2d9c811d94120c9d3591dc0669fb7e1,0xa517cf8e3f07940894f2c88333373572af028093a94c5b317a68587a0dd9993f"
  },
  {
    "at": 1788657764386,
    "totalUsd": 469.206941,
    "holders": 899,
    "realizedCumUsd": 52847.37439157964,
    "txHash": "0x9f8d683d3c57d4a3181ca3af5a70b814b2032b6cd7c0ecb6c0daba19e41d5bd9,0xa83db96a72b09174417a402214519f5df15a73da7f731e2baef873218baaf293,0x01f9dc62b5308c5085a1def47edecfc17f513a5327cf73457c6b9dc3cfd96853,0xdfa80a9bd2df643f9f34bf490cf8755bd78785a97c28eb4029eaf4985c277ac5,0x49c609225e0dcdb0221b7fc37ed90a397f5fa3eb7d37abec65f6a85a56d978a9"
  },
  {
    "at": 1788658692247,
    "totalUsd": 1618.743848,
    "holders": 1138,
    "realizedCumUsd": 54494.39629083274,
    "txHash": "0xa9c00df7a9721a335ec4f5c95775152d32ee71564ce1278a4a4590c75b827ef7,0xece1c8724208b9653f2e15617ad96d00bbbd18ccec5447e0370b2a96e2b12e59,0xee1bd6016cde3b2e459791a57b41347377888f1707277aa5090bc4190a07fcec,0xb45defe19d81abb010ec42cfa0de33bf9c1814d3554d2c60c74b48300039eb42,0xb72864a3c721e7eba911a92216cf28987a46df9431a0261b77f2ece80f04baa8,0x231be0df67494120ac06e9ced5376df6ffecf146ac8b3392d78c67a1141b40a9"
  },
  {
    "at": 1788659648633,
    "totalUsd": 469.915569,
    "holders": 898,
    "realizedCumUsd": 54935.98094546433,
    "txHash": "0xc12d914b7c41d77b72c7d71f6d83919c2be051515aafda730c17a6631077e94e,0x11f9d2f39eb8de825e20b6f30210fc3550045783b8af2f946b0e02870de801e4,0xbcad4f29b20a2c196eaf9f4929ab898597bcbc25da0cf5780cba36f697260ee1,0x500b018e7e829b7df658c3eebd6d4b87fef2e1bec94f2b25ff3a98bfa3343e8b,0xd14b5aac42b88590f04b1b06d890aa46dcdf6de7f17d78770b3c9f2c59c702ea"
  },
  {
    "at": 1788660686900,
    "totalUsd": 411.383362,
    "holders": 875,
    "realizedCumUsd": 55347.40737776869,
    "txHash": "0x2d0db61e11fbf89f9fb9647d933d3d0bb3e98fc38fc0a48a55b7f2e253ca962c,0x1b57f4dbeb4460d8d1fd3e4ea6eba4a402ab607410db48fab5e21ecd04a53074,0xfc0ec1df55f0f8806ea9404af7099733226346202c9333d0862de3acfa4cc2a5,0x0c152aeb4c541abf20c4bcfcc260e6d039c6597d5a49822a7bf315c0cb4e5550,0xa946db1729f2accb51818d8bb685504779b42540197a86608bde7050756bbdfd"
  },
  {
    "at": 1788661630429,
    "totalUsd": 335.622316,
    "holders": 823,
    "realizedCumUsd": 55683.09418436685,
    "txHash": "0x1832cffc52effe04e7ca88de7bcd70832e6d7179e0283dad476ad79e65f1d6b0,0xcf5828f5af66c4ce221b30511457f4da5adaf317c070cdaa00ef8109c504b983,0xa5e33ce2334cac6376357fe1690382fe82125dc6e9441bb2b5e9c34643c127f3,0xf5605c91e4e70d0c32af631f82e64bed9f1f5ae60030d187ff4b179d07c4e70c,0x5c14c4ee4b56381fd92873058141fd602a3c34b724ea7969a45bff610424af70"
  },
  {
    "at": 1788662950121,
    "totalUsd": 330.067365,
    "holders": 811,
    "realizedCumUsd": 56013.11800725154,
    "txHash": "0xc02448240c0ed0a3ba97e4dd2520d7e8d176b2e9f14c0a51eb05fd9850163e15,0xbcb8af39d9011d48bb6cc75ba681cca127a1e8e4f4ee6e26a9eb3336e010a7c0,0xe84355977cda5670a4d490a9bf8d283cda2cf16c0db1fbba8c498febb3f932e9,0x73ee24df4419e06d792cc23168df11db93e919d6c6b22451ea7b0e81dfadb169,0xd1471679870eebb518f25c26abc9a586f8be7086100d4b62082344d3e89f1e42"
  },
  {
    "at": 1788663918166,
    "totalUsd": 639.457466,
    "holders": 937,
    "realizedCumUsd": 56652.45824481128,
    "txHash": "0xbab6f6a320529a2f2ab34b78b459bbe2bee0efc4d24d0f653bdd5247a0afbb1d,0xe2e0b0b1ebc3cf694c808e4c27a626e1efa0e51898d90dcfebcffb852722c46e,0xdef04765b3c08c307372ccff645a00f45aaf45c096df6b5e0d5e623f5352bc46,0x8c17cd52bc01cd129a38993cf3b8e981994e51fe0dd266d26cf82057665da11d,0xf58ee346855dbb0c3c420187f2e721fb9c0bf237d44ff71718425aaf24661c3f"
  },
  {
    "at": 1788664846614,
    "totalUsd": 356.058549,
    "holders": 828,
    "realizedCumUsd": 57008.60163734738,
    "txHash": "0xede58f2ca2eab11a77d2581be712ebad34e1978aefaba78ccf83822b8b39bfdd,0xb57179a47ca51a955dd4f580f7c30f690cb7319fca538a6cb747618ba5c43bad,0xf2c91858255fc9d8812c57a564474dc2bc42334c373a9029cb8f68b3556a85e2,0x736c6535808269bf88b53b7b4b75f6eeeb8750c7242ce9cd65a1d175bd7d5a9c,0x34ace06cb7348ec1350045409598937ebaab70b2f6826a79fc15b4b7fdcb3d2c"
  },
  {
    "at": 1788666461465,
    "totalUsd": 354.56474,
    "holders": 814,
    "realizedCumUsd": 57363.15126217733,
    "txHash": "0x3d496242f9253dfc43f0e261615736dc92b7e0ab74274d372efafeeeb7e2b02a,0x27990209cb08a54d6e71596496e4dbf080a32447fcec73fcb309dcff711f5f29,0xf1b98b377324f078c95b9423e740286a58358efb2b854f39fa2c55496374a808,0xc5521454b34a2dedb285640376ee0f9e513d3a52f3aa4e5eba54e5c41bca184e,0x4e68915304df06a0865ed7a47283af1168845fd63fccce2af6df57650ba7da1f"
  },
  {
    "at": 1788667426371,
    "totalUsd": 855.957145,
    "holders": 969,
    "realizedCumUsd": 58219.006393669595,
    "txHash": "0x515d55af71a1b35738e49ba245b3d47119f3357221b968321f0b210b751afbbc,0x06ec372a39d7712c9356764fe4efe0fe62b19d720a37e64e00736a363f4535f5,0xee7158cfb057aeff5e8e9401b90c3df6e3a5d85731d55fc43d3f486cab31cb61,0x18e27288713844a10663eec9d724f2e08846150c112b03022b13e0c1cc471296,0xc8dbde9f62792494f204247b102c935ad0b09a9f6a41e70d537db38b741826fb"
  },
  {
    "at": 1788668441534,
    "totalUsd": 463.593133,
    "holders": 864,
    "realizedCumUsd": 58682.65129476009,
    "txHash": "0x20f389f7810dd96172d541fd801373eb39cb93e23c7edcae200305f985ce6e4c,0xd8d66a0cb219bef2c3a76a74c0b36d3509a10b3ba219e587eefc94ca0509e1b1,0x3788f65359027017c50c138a2843c26793710fa73b273da7b8eaf553405337cd,0xb5f1f887d6f240c0b84972b9c92c84a7b0640964d4f15fd4c8f68b462253bd18,0xa43096d6aace3b4ef0ddda35b33a0454a89ceb6cdd7d647e9468859f225521bf"
  },
  {
    "at": 1788669401859,
    "totalUsd": 428.695816,
    "holders": 852,
    "realizedCumUsd": 59111.31145181175,
    "txHash": "0x49d1e3ebab714075d1fed81846f76e5ff3810157c1fa25c01e5af20036533a58,0xc484833307001b4028a083de9c7acc42b3c400f4aa7c64409c84f0e54671fb45,0x21e2197eb40bddf3a1684e20ef7b8c17dbe31d6fdef791bf7995a00527638bbd,0x47430158abcdd3558ac87b2f5b28f16fe33b92cc62e2a60c10b1c8c80c6a42a1,0x7cfc2aa8581a6799b247a68db61d8b0d843d6e90b1c290c31e07b692d2b2d977"
  },
  {
    "at": 1788671139365,
    "totalUsd": 337.154853,
    "holders": 797,
    "realizedCumUsd": 59448.5015874613,
    "txHash": "0x3e934f1801c13dc3db72045ffd8d04d60967166d36a7f449994e90165e918fc5,0xe942bba1f345ad5c1f8e6a8ef9de517022e197804e1859e58246be5dd59ca4e2,0x907d744f6a04a3840a243825c058f33fc7cceab286610d6a69491dc1837d399e,0x6d9f0a4b9bc8e85ed3e94b00897e7e0c133c0ac72ee845e5aca4006fdcd07159"
  },
  {
    "at": 1788673119752,
    "totalUsd": 329.455798,
    "holders": 780,
    "realizedCumUsd": 59777.93203875611,
    "txHash": "0x751b4131610bd8f77c085075ce98b1e6d72ac89f1b73e3cca96fbe5041ffb96c,0xe715d9a94948e16b38191a1a5f0cbb03a891ff0e7165d95458be4a80c7b2c600,0x527c62c4905c2d0df3db53c46b0aca439b2629645719d8c1f551480602a9d847,0xfd007bdfa5fe22f72cd949f4045fbb20146e09c64e10ec6690b3efa0b6ca8c23"
  },
  {
    "at": 1788675520004,
    "totalUsd": 300.732434,
    "holders": 749,
    "realizedCumUsd": 60078.65987615247,
    "txHash": "0xfcaeacff8d86727ba082aca58f3ca52f9fbb1944dbaf4112092514e81edc22c7,0x30dff75afb68c63ec50203dc6239e840d58c4bd56a9af11e20fec850ef89b70c,0xdc64221e35bcbcbcf9ad0b397cae20c8bbbec7547483332163abd2475f4ccd1d,0x6efa4af3e9e6076e02eae7eaa647e7754dba906017a94358da382533528a6046"
  },
  {
    "at": 1788677368431,
    "totalUsd": 302.104948,
    "holders": 745,
    "realizedCumUsd": 60380.758506636004,
    "txHash": "0xcdc20e95d38a09ecdcf08f8a589e978285661ab9e492f5f2f85563cc5d8ac02d,0xc042ff5303ec6104e732c2e1523228e34a049d968139555aaabb714ee86decae,0xd51ccf427b03cdbb0d69eac4308e9239753278573f796d1cabab2b50c8ce4ccd,0xe0bf5be5f6e971a36a088e29fdb9c48b61f1687d54f749fcae7d72b9c3016b4f"
  },
  {
    "at": 1788679342451,
    "totalUsd": 410.995112,
    "holders": 802,
    "realizedCumUsd": 60791.701098727506,
    "txHash": "0x32e864da6108ab32260739792b5f37408efbe397ea13f91dcecd950dfac80a85,0x196ff04c51aed503ac50a011e8e1527310fe9dae94e7ca9afb6f1b20b518ba66,0xe0a99752d3069322e1df224ecae1cfbadd7506b9dd27347eaf039198949ba9a0,0x07374262971d91033dba4541d0668211cd1b4031cf257ad566b7000822334e17,0x2bcf7d9c06a3e2b96503791ca15c8fb20c950bf33fd590156c2a25b19584cb8e"
  },
  {
    "at": 1788681376642,
    "totalUsd": 372.282024,
    "holders": 785,
    "realizedCumUsd": 61164.0038807275,
    "txHash": "0x9a12c769e2acb1641ad3ff2b2b49d7ee21563f3a40e9c32084738a2944ef2eef,0xac28570ec556c1102414187e0f70a4dd04d3094da1b7ff7cbd3c30e9237a2599,0x54cfed6b3203bd90bd19f8a9f7aa4beb1235ecb335056788ba1bf556e229d544,0x33165bf444da1d0aa07863ae9f2235f00bb146f56e2552ebc822c1dc45d8d973"
  },
  {
    "at": 1788687174395,
    "totalUsd": 340.470046,
    "holders": 771,
    "realizedCumUsd": 61583.29887968814,
    "txHash": "0x9ad98aca7bab18eee19112c3b8244bad363a606ee938eca1d9860fc29c969714,0x9aa682d9af12b292487aaf0ca6b831ddb94009894872f68620d9f87c000f053e,0xe78ee66e7dea1e8db99025a4eb4cc8f572b27bdcdc1023541003121c1148eee3,0x33ab67eb54c67e11d36434cf1822742a655ae472c080f1538069425c1f37ec4b"
  },
  {
    "at": 1788692042596,
    "totalUsd": 313.699091,
    "holders": 753,
    "realizedCumUsd": 62001.61324709243,
    "txHash": "0xc38ed04b47e323a099d8363555a3b97244565d9ec711fac8fd34c83ce4ff6084,0x2e74cad0461c4c4422a3b45e8b3533648bf2a1a78db99c97f5b03e79c7c5182e,0x63bfcb4f0aaea9fc1f8bad83b1c31701259d7659595c28848bfed8ec42d7537b,0x80c45c6c1d93177aa01b024ce1040cd492a6b7eae8aa32dcdc179713f9b9669b"
  },
  {
    "at": 1788700603318,
    "totalUsd": 315.400103,
    "holders": 722,
    "realizedCumUsd": 62422.06971026169,
    "txHash": "0x4e67c16df0fba9533b0ee805d5e35e8f2d8bc81c3915a4d36fa525cb7efb63f9,0x54cbf6be1d3ea770279b06960b396224f22cf7d70700d3ed9666c841c6d89981,0x304f727dd4e3688ed81bf6ba22c8af88bf21a45fa5633833b94f8071572cc3f8,0x1c242c07d15cbd2b4df15bc52ec2e9b4528c2471cee1d299ae145acf69838c8e"
  },
  {
    "at": 1788711597946,
    "totalUsd": 314.718589,
    "holders": 679,
    "realizedCumUsd": 62841.6076505216,
    "txHash": "0xc6e0665ef99c5a77321ac1b016892f1bd0412c86bf203321b34af5e063fa8996,0x5eb25ee226e6cea4847a78a7393d8ff8da94f76780c9d42a5fd9907f5b4c7cdc,0x0917e6929d23f95d7b1bf5d88ebb2cb4ed858f48dd4867570a0ad6b4d47a2e3e,0x45a976a94127c82d2cdeadb73384dddea14abc8907af49404740f19c5e26518c"
  },
  {
    "at": 1788721392680,
    "totalUsd": 302.797491,
    "holders": 660,
    "realizedCumUsd": 63245.2783550688,
    "txHash": "0x8965c3a9de08fc127ac58d522b0f3aa866a7da84113b7b2e830e4f69f3f0784f,0x5c95337bf4a4e62eb29c857eaf86f5aed336e8a442d1960daea596a854e226e4,0x4463b7949002662451176fd1286841dd845d65eb3a2df03cd4d5bb03baf6d110,0x2903c7bb8d89dfa82108d3e923bdd267a72251b5fbf9da0b117585b14963036d"
  },
  {
    "at": 1788723113824,
    "totalUsd": 369.733038,
    "holders": 685,
    "realizedCumUsd": 63738.21475035682,
    "txHash": "0x958471c788426ead40da8f73612e87c8ba1487b94a70be39b07730e7d46f7760,0x2ae3c5087805a478dd587fad85c91130c8a9415049ed4c2c45d1563f0ce8e627,0xe5735ae36ba28cfe35224a687c5523bac4d51044b7a18ed977cc459b3e6b8396,0x39abd128592d59f3cd26b61efcb3479ea49b5dc3b8e9a2fc4eb29019a6cabf8e"
  },
  {
    "at": 1788724046167,
    "totalUsd": 419.671223,
    "holders": 705,
    "realizedCumUsd": 64297.70857460612,
    "txHash": "0x348934e47175a70679aa4e25ba89f51d66e58298aa680172a632ebb5c4600d26,0x377f132352f9782bf94979e54354bed6413506b835a6d112647e5e6e432e022b,0x5ef0f8a158940e74c651281ba788e5ed06e3dc39f785d3bedcb327ae2c974d03,0xa372b27e557b9311972408adb9d61f364c431b744466d01e6a32d42b77aa1002"
  },
  {
    "at": 1788724995647,
    "totalUsd": 371.932521,
    "holders": 691,
    "realizedCumUsd": 64793.71490060612,
    "txHash": "0xc648e8f54f237f2acf3541259eb536159ca14ae064c7821590288b351a62a000,0xdd09b1e4f74687286589b01e38a0a86300b6443cecfdaf5261699048d32260a3,0x534cd5cbd371dab7ad0acc41e7c4c006e40eedcc20b090d46058c585c805e481,0x3ba29535b88cf86ca415f641005f023de49b91c084f029874f4a3081ef4cdd1d"
  },
  {
    "at": 1788726953538,
    "totalUsd": 332.510303,
    "holders": 672,
    "realizedCumUsd": 65237.06770623791,
    "txHash": "0x14ab81e14b2328c2c21e31c8bf3da847d635e18212c3bbb44bd1f99bfe881edd,0x02685c8a86cb3bc745deb5635998212c314829ceedef8ee374cc7dd50a4b031a,0x59cfd5ff2c5bc1ab62516a9c92eaceccf311697a313d0717f4f3f7322952050c,0xfddb69be76d58a8b9e191f1a7c07692e6c9a60d64d57ee315d6c2137e83d5e2b"
  },
  {
    "at": 1788729455120,
    "totalUsd": 299.612388,
    "holders": 655,
    "realizedCumUsd": 65636.56272778953,
    "txHash": "0xb8a2bfc002daaeb8c931e2dd178abcd02f37d7010033e8f662654a383ff79925,0x98bb6eaf5e0e52f70e788f4cc0213c889b90e7cf8afa32af69f5332101db9bcb,0x7381747e67c3674c0a1c3061df331186c234af6794f874a0ddebce4feb2166ad,0x1ab2138d6612554bd18a08d13528a2b3a8d96a9e4d570be20ab54d6ae4b1d556"
  },
  {
    "at": 1788733513325,
    "totalUsd": 607.713188,
    "holders": 758,
    "realizedCumUsd": 66446.68620578953,
    "txHash": "0x556b50c3f2f3da4d0ee5111e4064ba07ce1fdb38b3be0791c3e64a78778e3121,0xacb9dad4e7f786d0130937d259347434839f342b184549a2be51f73b4ea87271,0xca82cb38b45de00dbf440375ccbe4c42bcc727c5e3167f97c4ff0a27a4a53bb3,0xf78c0ef5796e85a309fb1a9bda9e686e4f5909918a3f96855144ec1ebe3536f0"
  },
  {
    "at": 1788734483842,
    "totalUsd": 1547.534144,
    "holders": 887,
    "realizedCumUsd": 68544.27018534658,
    "txHash": "0x6626352500d2f56e3ba5e97c9e0d0b4eb964e2c0f87f8e6b3dbd9f5033aa9813,0x04c6d5a14a46099018e130316eb8268b0c8d25d7c6c824f806c0f8a3311a04f0,0xbb998e14ad4ecce3ba5a0159fe630b0ef14d4b395ec2a08db927f77f6c266838,0x08f40e08358522c449db099f8192a38ea4b26979def3086144f8cef113b815a7,0xcb2f354e6af72f6ed3eb15d2e505020d7a926377a4a03558e8462dea15d4ef45"
  },
  {
    "at": 1788735436895,
    "totalUsd": 1393.860343,
    "holders": 879,
    "realizedCumUsd": 70489.14352934659,
    "txHash": "0x62997fca0b567b101a85ea2f35ffb6ff9ee374c076219a595d86b78c59388983,0xe554384d8d65ced2aad902813acc136bf2f94d097393fc8260399f2de9f0bdc3,0xbe6a079417e2c7225f3d3d494052d249b0246a8b2ee94bdf1b9bffbfadeb9fe4,0x148ac78d119831859057489406b3b1147c3ce753d32093600f8b97c52cd231ec,0x5efcffb434277690c26651313d63e4b26740378b17ad4ef15c6dcf077271a2a0"
  },
  {
    "at": 1788736412638,
    "totalUsd": 986.933005,
    "holders": 821,
    "realizedCumUsd": 71684.5018203466,
    "txHash": "0xd4cc9c6aff3973116d35e3f70f89d056c652c492b131eba0764e59be81832f08,0xd2c25889a7f289dd0cad9099a84cec7a4ddf537efde7d49699eb4598ef5d1042,0x35f9c461c29a2ec14e8b4090485eebf74f64e3a55b8c5fa58beb1dfee937f8cb,0x2429b3a3d0c5ad0a6577807ff8dc0b7e1fdf216ab56e477c6f037f2130cd4f18,0x906d990abe504b7e93a0cd968fde97390ed885c7200d67e3605c21d1387ec200"
  },
  {
    "at": 1788737363350,
    "totalUsd": 871.734999,
    "holders": 804,
    "realizedCumUsd": 72846.82203734662,
    "txHash": "0xb4935fc44c27de49362ab8b6b3045cef420f3b6bc85d16a7c09c81570622fecd,0x1238f04adb5575f11eb67e278cf167ea92daa7391ab74123e4d73c6ada6fa563,0xbdf018a34422b5474dc9f0232c0a9b92c88f890fcdc3600db47142b06c409b56,0x5c352964d922517c2aff3dafbd9d460fb007d34cb965b743c460c4a80100858b,0x0c167a4939b8305e8328e6af1c316c3d35ee913780000dffb0d17e6f53033738"
  },
  {
    "at": 1788738306588,
    "totalUsd": 1063.521588,
    "holders": 835,
    "realizedCumUsd": 74264.84148881039,
    "txHash": "0xaa8071500cf049a719f98489ddfda4064281df26e1a832c83ede68d73cb9e18b,0xd586d39f69b5a481df9717baeef61a7fc177bc20061e3e441a0e87d87231aaa8,0x97db8bd7f7d5442f5a148b773ab9a4505bea3078fba9994a7625f5e3c4b1f5b1,0x12a3b430be91f73b022e4ded4c1f60a45613abe4cbd082527322dc6325a71d8a,0x9273e12bd92c97493100d0bbb682b064265cf818783175fe376a1f85bdb89617"
  },
  {
    "at": 1788739271746,
    "totalUsd": 1178.002439,
    "holders": 859,
    "realizedCumUsd": 75835.44319754712,
    "txHash": "0xede9bf98608beffa410677193719a5c0a57516db123152adfe5d9127ec21671d,0xed673c386ee717fdedb8071904eec54869b9d5f3fddb42c7457cc97e3ceb12d7,0x2ee11fe04daacd4257b6a45b14ffe9fc9368d0ed8af2085a8ce3b4237f0cf633,0xacb6b1c614d75aa4bbfb5625666500e497104222909280368c3578abc044f5c5,0xa0fdaaf35eceb7c0aaba4ec3f6d4645db25ef5bd0eee7fd18517672948961733"
  },
  {
    "at": 1788740238706,
    "totalUsd": 2336.521587,
    "holders": 1062,
    "realizedCumUsd": 79224.8883177447,
    "txHash": "0x53379bfc725249e023ac22992aee1c54ce961841bf0f96dba6050376b871160d,0xead5588f77f6049e66d5fa645363512097df4f7f6034479a194eb21238750e80,0x73a958e7cf17b743d5c4793422c4bf41f469f2d673911086e270bf2b95f8d9c8,0x7de5eab82a75a7ad1918dec397f2725a5563c28a7633c75876a4aaac0d91f75c,0xd453499e27ab5bc7c37dbb8241a5e770b24171fde8e6a1248e2418d9c0b6f080,0xfe7734d0c64a776b8d52b563edb2d453d94e7def9d6f2bc4158ab32a8beeecc6"
  },
  {
    "at": 1788741165665,
    "totalUsd": 1784.813807,
    "holders": 1118,
    "realizedCumUsd": 81331.16381174469,
    "txHash": "0x7e2519cf71da7f8740bf794c08f2344879b63d77bc352db3e34bb610b9a203a2,0xc258e2391a443b6c49172b5eb5113ea7b3a74d6c69498d1cc14bcfdc5ab9b905,0x96b3d7c16a4f66ef0cb9dfe04b80a9f2c8c003a54163fe36df93f32fc667ac32,0x79e8b9531e7ee38e945f6cf07e76f4bd34b5fa35bcfd42d7f9e51178d87fd6f1,0x6730be526cd406578eedb46fe98ad7da4f8dab7c9803d8ebf53de48681679f15,0xda2e41359d574c84150b6e35302c9538c9c9f9b91122da8430792ea1a7cd2098"
  },
  {
    "at": 1788742103212,
    "totalUsd": 713.426806,
    "holders": 920,
    "realizedCumUsd": 82282.7300335049,
    "txHash": "0xca690ade45021f7a98bc88bb53a6ae6e5cc00ef7cb5aecc36ddc3f2d5b7fe5e2,0x8522f69898826ab5962a310cc1f4bc348d213eaccaeb6d61da024982c414a50f,0x9788d37f497b6f59a1bdab11bdd863b42d0f7a1d84ea02cb86c3c456ca72e68c,0xfcaf62454e01526f521ae4b91d9e0b4adf877a742aaad7561c442bd2f0f4329c,0xdc30551c683182bdeae849c6406796e980a9c35bed89369274679079b21f1c5c"
  },
  {
    "at": 1788743027912,
    "totalUsd": 812.677579,
    "holders": 945,
    "realizedCumUsd": 83368.40386340057,
    "txHash": "0xfb129675f56e7def24bbdb90499de5ad3ceea94bcabc6238f23a1e9a0a63170b,0xd59e0ef468422d078c2d3e808bacd8fc62c7f4fce5f3c541148e52fda012170d,0x3815c4d2979f5a679ebea70f03d62fdffe4616f677e8fd68115e6967fcd76812,0x220bba37aa550a224175e807972fa8a542286eac9e943e4cf260bfa20fd8123f,0xd1f57d707e778efca3c2c1795d7151613b55e92fad0c66c62a549ab36c138a05"
  },
  {
    "at": 1788743972840,
    "totalUsd": 566.593077,
    "holders": 899,
    "realizedCumUsd": 84121.71579040056,
    "txHash": "0x7be337dcd4f997490e764ee13ac5fd66c90eb84b30c88a30734905d9f5a570fd,0xc2cca6b862aa1ee6c4a77e7053182c36cc0e95e2e8a0caea0b67104d636b285a,0xd37ff07e62eeea82de8676582fd4fe531fbcd9ff10e730c75235faebef36fde8,0xa840d92b57d28109453071a117ad7e0ab6b02c46ac6b9770b83e4b73538891cf,0x3743b77a8a9dd8b4dc453140701608168b92658a406d38e01a00fe167972d343"
  },
  {
    "at": 1788744930714,
    "totalUsd": 622.867909,
    "holders": 922,
    "realizedCumUsd": 84952.20284343405,
    "txHash": "0x062bf48222d9b3258ae28483dbe3ccfb258f876e3133bf55c5c1709e1c209a75,0xb9b04c035f92c8896ef98b796fec63919683b62aa462e427bc1ca08ebb6f5c49,0xd453af8f4b185e9f729a01eba640d36a9d880fa246717e0aa9ad0966c6987806,0xf36a4feedd7f8665f9b45b56d00372df2e1a751cf0d6b24981beb74b399aaaf4,0x31bb72baf4426b6d8bccc935f1dfaeea3b8b436aabb5bfec27ff494b4ab2178e"
  },
  {
    "at": 1788745953494,
    "totalUsd": 841.523464,
    "holders": 971,
    "realizedCumUsd": 86074.33980943402,
    "txHash": "0x3083bdf06a25e677c369bb2b7dab4b3729f964f43546f0c01755283d5bdeebb0,0x77e7b52d76781924540db6725fa988f4086a14cdc6495861cfa43f4d8e3b4983,0x8f337c1edb932b8a165d3482cbc92356033c076133c397b4f81bcd23c1be0481,0xb189c8802af771b8fa6b6eb3f12daff58a0265d8399dc242c1f5afda87da751b,0x5a2c68eb675e78c70b9ead8f7932c7a9ba051e4df3ff3b751d183880eb5ddd53"
  },
  {
    "at": 1788746913428,
    "totalUsd": 3929.985415,
    "holders": 1316,
    "realizedCumUsd": 91333.0274927558,
    "txHash": "0x2115a94c9ce2c4942560dc89a005ae5263641cbbcd24102895cb52692c6148cf,0x38df162460ac2643bd8369beaee599123828b56505d608c01c7f3a4d255ffa86,0xc7585ad1c02d20a42c20ff07584692c72c0775461369c9e9caca4c84935d1bdb,0x93df3b49867549c6c70c20dc162d377eb9a74326461125caac9f7b42672d57b6,0x80d830b10ff37d3ac9261ccd6ecbdc19fd8815cc75d130d294fb473d240f7876,0x514ad73d931ae9bfab7b7f078ba46551e74899d9c9f6966bfcc35fa1a6cd6136,0x853203e94ac3f4d07c5889b3fb2204774deea1ff65360a483287dffe931c6c87"
  },
  {
    "at": 1788747866696,
    "totalUsd": 2126.424669,
    "holders": 1188,
    "realizedCumUsd": 94149.05624493706,
    "txHash": "0x22d8975389a71f71979563b96a60bb023345b86f792c0ef5147a235217ecc2c1,0xdcfb907d3445bec4b0c0e0729839c04b854a199d5a429696eb337e48406387f4,0x8f6d9e3ec2feae4b4c78373d586d277c958601671f70df88ef685c1fe182f67c,0x903f3017a09ce90185c299be6e71eac0c6f6ae04498456a211efed1207eeefe3,0x32c485cff079c1efa4dd1f88217973f7ec9204f0e6d3884799277f726391bfbb,0xc6062f6e898b8b33454730db5cea98d49e8223de38af160ea833d9edd1b91942"
  },
  {
    "at": 1788748816356,
    "totalUsd": 754.459674,
    "holders": 961,
    "realizedCumUsd": 95155.3739519371,
    "txHash": "0x2aacb42d4b1b5f67ee05779b5b47512cbe81ab05783c6602fc02be822cb85b89,0x7621110c74d8c4e27e05d35eeaee8fc7d549ec31fee095a1ffa8c1f398a3d4a2,0xc251dc23ba4bc59ac5220aaf1ad837ccbaa55917a033695b7b066a6ef096f92d,0x195737bdc8b92544de4c34cc3c467fbb5ae116c0687bcd7d3ad20daa58ef20ff,0xac483b62f480e6ad9d8fa1b0e51557213a347cc9e8f5f1099454133319ded995"
  },
  {
    "at": 1788749751296,
    "totalUsd": 968.26821,
    "holders": 1006,
    "realizedCumUsd": 96790.05420210201,
    "txHash": "0x9bf1dcc297fd2cd388a269b169ed58f9f1b0130166fbc80229e034603b9d24df,0xef61eb14ad9149fe8e9d2da80f7c7e66f6883250278c46221a5375769f660aac,0x3260fd79054a078315ad2cc808bdf21e3521f020f0be908fdfb44d6b02d4d588,0xef51e5d6818cd9b9b9464535a64f44ffd58940509ea700787ad0aee55b1ab85f,0xf466f0ec940525f955df67f0b750077110a3c70c16baa779ab96d76af3248af7,0xd6b61620ac308b8e2bb35f51ce5efaa3a3384f65fcd6b8165f247a47b73e69c7"
  },
  {
    "at": 1788750706435,
    "totalUsd": 1164.846148,
    "holders": 1051,
    "realizedCumUsd": 97999.55772710203,
    "txHash": "0x8253daca19e878b04c800bc7c5aac1a98384b78dd3418e3b29a5a139a1cd79ec,0x831d7aa99a3e1d26cfb85a51004062fd64ec158d67a5f5b52e77ae8184a434a6,0x5c16d98691c86bb3411df288dcee62e2e647dedfe94d93dae671da3ba74c9b2a,0x92a3c871d6a330978c4b2d705ed2b38b9b9166ef03ef8e7906083227fea53a8d,0xaf760c0c80796c16ac3357e4debe14b5730b45230a4c1cb32ed80ee0004e2d82,0x59298a96a4ccf3c70b2f454dd97154a98a54bc05cce94970e49b28a11083dcfe"
  },
  {
    "at": 1788751681416,
    "totalUsd": 760.517144,
    "holders": 966,
    "realizedCumUsd": 99013.46917169624,
    "txHash": "0x5355e331e7b6e9641d1e1600efa36795d69c0d0b53c03d7e3d9f2b7f7d30a5bd,0xbf28f513474eb4e2eb90d49d91cb2d2c4f1c5d2a666f3256a2a20c90a4a79cb2,0xf52ff462e2834798a38cea9a89df53169ce90b345b1cf8e495eb5b5136ea338e,0x8fd26990208ce60d4a0fb5a5fd202a9c0e43b3db5e8817617ceeae00b1cd0327,0xded5ba7b55bd22a6aadcc1c549976e8577af2302ab015fa9df7a5bf736f3b82b"
  },
  {
    "at": 1788752618169,
    "totalUsd": 425.372079,
    "holders": 200,
    "realizedCumUsd": 99652.53643069627,
    "txHash": "0x4e2f9362d8cb4bcdaf9b51d834e0d0266e39c71e9ebd37b6953117f33b73b1d7"
  },
  {
    "at": 1788753564392,
    "totalUsd": 512.923904,
    "holders": 906,
    "realizedCumUsd": 100264.56735969629,
    "txHash": "0xa4556c03035a82ec299c7cf5972e86470ffa7f8408d2af936ac5da16fc744e32,0xf03d428e8a42a91bf0ff97e10174dc247961b47e2a12792023d3e058656010b6,0x7de58ae6e5fbc92cd58484fb27d677bb65a45b0dfa74274708d296badf027093,0x8cd4a5f788a56d2dcc0a97d0cd8c80fd33214d04240b80e89118663d7cefea05,0x357dd00346180c44cd008ea3c515e5f4785d82aced7bb0b30e013449ea4dabad"
  },
  {
    "at": 1788754822288,
    "totalUsd": 405.455125,
    "holders": 849,
    "realizedCumUsd": 100805.3521696963,
    "txHash": "0x142138673d5baa86d2bdf2827b985cecc56f53b7ee7685a6929c1a40b6eb0e71,0xf578732ab41a7f759a68495ef84afb6d9227921df12919e5826bc6149693e81d,0xaa17e5a811168f16cd6e6f63a6476c924c0eee7824821f67891ce59f88b22cc2,0x4a6cdb0c2b3eab0c4af1e19a35708acc0d803199cb59cccebe7ae73e7532e5b8,0x379ccd077dd91c3dadacfcd2075766977490feee9e5398a334620737a20dc4da"
  },
  {
    "at": 1788755935364,
    "totalUsd": 333.58917,
    "holders": 793,
    "realizedCumUsd": 101250.26641869628,
    "txHash": "0xfaf2304e082cedcb3729f391028085db1868e15e27f758ee8764b9cccc831ef4,0x8c94f3d88da277f7cced910628b887ef79711ea9cfa6308983ecc4f69d8d4457,0x06f9c2a0998fa3fd25c8c3b4f1452c57f7d16578ba446f6e1b1e8094d62aa6a3,0x9c9be9df4f55c155bb738ac0e359b2d8598998dd837736507d1b74acd82bd7fb"
  },
  {
    "at": 1788757129413,
    "totalUsd": 341.830672,
    "holders": 793,
    "realizedCumUsd": 101757.64173080534,
    "txHash": "0xaa036f63a440300400709a1b536d047267273d3f599e14887c0b1f9fbf0c4cf1,0x1fd7d41689103955937c833f2f6f9e6a918f1e0aa921b6c263e09bc92bcea887,0x58670e23375d9f2647af3d735722cddbb7babcddc0eb5d11bab5014b876fb6f1,0x3f657a7b1d59e4e716de9d2c85043270e49b380c5c3231d374426d7eb0f5e64a"
  },
  {
    "at": 1788758061024,
    "totalUsd": 588.212225,
    "holders": 916,
    "realizedCumUsd": 102490.01431765247,
    "txHash": "0x6a11b132070da176aa846cdc600dd8662c995b7b4c600163cf38e714a76899c5,0x118ff6be826a5ca0fb4193db35247de3f8a57fdf0a6288275fc6dfebb0a7c54e,0x913d94fa4710b176d85b4ba5318730da8c03dbbd3a410ee98b2e30e5798b29a2,0x45119f518c25eee5164387143945652434c81c822cb7b79594ab6fae2d620ffe,0x9d84ead49861e396922f6b7d72f37ba88aa8b80dcc204e9c35e4977fce6c7f57"
  },
  {
    "at": 1788759004160,
    "totalUsd": 652.047372,
    "holders": 946,
    "realizedCumUsd": 103359.29639322626,
    "txHash": "0x1b894c5af742575af42de535ec0d9a2f5911220d6dfa418e2ad99774b19f381d,0x6f606573b8f5488f72bfbe302685fe8ee46dd7dd6250356064ed747ff44dad80,0x904cee9ea902207d71c5566b69c409013fd4b7dfade734342b68abc29567ec59,0x93472cf4f39b3e85de7fccd76a56d8b7fbd588434d8b5635234f841c2c2b15ac,0x88aff1655b3ea1bff61173ae026b0d718433c2560012936a86e695ad2f95a729"
  },
  {
    "at": 1788759932656,
    "totalUsd": 326.011595,
    "holders": 781,
    "realizedCumUsd": 103794.42027324853,
    "txHash": "0xb2608c3db14f67a534d5c0cd9f3fa45a612d703260a94f27deefcabce603a228,0x6ef480798dad59e123ad2df3d8ce4a13d3ed91d550f295d19a9bf793786aedaf,0x1ecb504298d63c53ba37714a7d687dbab3e1ee188035ba0503ae2b6528381aa5,0xfd40d7ab7143d2c85f7a40a426e65e698bc483f32b03630d6f0ead9661b2ec6d"
  },
  {
    "at": 1788760852552,
    "totalUsd": 344.152507,
    "holders": 797,
    "realizedCumUsd": 104253.25757455727,
    "txHash": "0xa0259456757d26d5c78aac690d3dccc460e9456d2aed131b8ca9cf2bdd4dfb87,0x8189659185a78d9670519e9d3003e703ca5f1d74728b8bdf81a4b2351854629e,0xf28487b85e7a360c9dfd3e8864d5bdb21899473e2f95e8536146e1522170a36c,0x23105052023bab74f21c3c36f7eafc620b8fe09992a51ef5f8d8b8a09e692f89"
  },
  {
    "at": 1788761779538,
    "totalUsd": 638.27203,
    "holders": 948,
    "realizedCumUsd": 105103.86316082475,
    "txHash": "0xb565daeda6eaae99906246993b2f090bfb5f09c228027034fc76d80c4647c2cc,0xff4efad35c5215dfa0e692efcacda5cebc0b35396ca79d0c45ed5548222119fb,0x9a9d882f456b001adddbe75313c576cca81ba1496162d1126157f483a2bf1084,0xbf53590687aae610a1fc29e31adad7c1f1a420ccae8ed569cfeab9b50dd20062,0x62b87d32116fd4eddf275fe65ffca2121dd9cd90027249b2827c66ce6354f30e"
  },
  {
    "at": 1788762854619,
    "totalUsd": 314.782301,
    "holders": 776,
    "realizedCumUsd": 105617.42913382476,
    "txHash": "0xb5e0aee853123b871864440cb804c54314aff419d2c423bd1e9b48a6301a422f,0x3c7192a3a0b2002441b36702added12affc41c9ba468546222537276f2df33e9,0xc2c7ae0c91f04ff397f68d0eca68c0776c177120cd1d886f1938fc07268795be,0xd8c7f94c73449996d0c735d6713a9404b09e3790140213f243e90a877a2fa68b"
  },
  {
    "at": 1788763814444,
    "totalUsd": 473.786612,
    "holders": 886,
    "realizedCumUsd": 106155.41535997753,
    "txHash": "0x3a98690c873e5df4e8de42027cdb9fa3b436fb6f38c454ec9040cbbe7b40b9ce,0xb252098ba3815f9817a4156d8fcd45d2f8f66804e248543adc7b91dfa56a1a28,0x264ce87ba678d1f84a407ca80ad91b3e20cebdcaabf55d7629905686d5fa6346,0x3b34ec7d6e51651634e39bd303a66fc359d6cc65bfd8592531037b44a6944fb7,0x27cc809a057da6224b836609a617c9442ef242608a4516ed7a24e7bdd11fed46"
  },
  {
    "at": 1788764773498,
    "totalUsd": 354.838193,
    "holders": 808,
    "realizedCumUsd": 106628.75375897753,
    "txHash": "0xdff9f12da87fea646499b741b8790bff9c1ac66c8bf3877d2d5bab80e0ac4002,0xde6e961f45d3e8f6e4063c50aad4b343b5c947b22ad3361f250494fdfc7ccbea,0xa158ea9753013dc4cf7f972ccfe6ec06cfe16acb6a90aa3bbf7e6017b35014ab,0xcf9b8427dce398f8cb84a85266ec15a19695020f46b603dcb14bdc684b0d6502,0xca202c4f84859e11f1bb13be05d1cf6298677847ad3922145d65c39765d1636a"
  },
  {
    "at": 1788767587519,
    "totalUsd": 310.345408,
    "holders": 785,
    "realizedCumUsd": 107042.55935497752,
    "txHash": "0xde6a1483825fdcef59c8560b73bb5661b3a3ea96ee5eb146e71940ebd08a0b60,0x7cb325a4dfd8e9ac90ffb582f6e6eab4264dfd4e7f9820d0dafb3944edd2fc40,0x032acc66185dcd61575825479d0edf432dcc235e3e3af879185726b7bc636c6e,0x64c0e730c460e85ec9dce27e936577266e18d3e972c6099b6bbcbad652a30c3b"
  },
  {
    "at": 1788775568301,
    "totalUsd": 343.432707,
    "holders": 809,
    "realizedCumUsd": 107500.45524497751,
    "txHash": "0xf4337ada79d57fbcf7e6655f019ecc3aa6db8fffb7084eb4a9138508fc819cf1,0x7a731fac822947ac3ed77782d6123b116cfad0bf3d1007d129479d5b8cd1500c,0x71d3fd6ba3ed4eaf02e78216a1b7d0cd38326a556a7e807106aee53588e2b08c,0x52474f2067276c7e397f7a2fd934cf8dc37955a208c73b2b5b82b6b04a95fa18,0x1057346cf432bc5cf30c5afc982027aaf2f4f8208206b245c044541ceb4a6239"
  },
  {
    "at": 1788781682201,
    "totalUsd": 306.901997,
    "holders": 778,
    "realizedCumUsd": 107909.62315404677,
    "txHash": "0x0891e68ef15715b9e3b30487ad5bc1c697c0f4397b3c4037827f4162d42fefef,0x701777225c0b420a9b4194c99369d6d5f0c1033873b56b398a428372d3fa3ac8,0x0f54461cb894fcf6e400d66cc6b82b1feaabf4d071d3d93fd66fc0dca2e0e967,0x3f2178f40add1ada6911cc467c1a8bd17176eb748dce78b6bda28a7da4274a91"
  },
  {
    "at": 1788787632010,
    "totalUsd": 329.694624,
    "holders": 793,
    "realizedCumUsd": 108373.02531289344,
    "txHash": "0xe2d7d6c8ebd4917c0b49868b20ace23544bae50ae43b6c61a822d4a19c5e847d,0xd5dab972fe7df8fac09267cc58ec9c09a68d95de8e5068c6c71baa2aefb5c4a3,0x8d7b8a70a86e8cf52ad7667ce9a170fe7a100007a55637e8b38a0e5da16a693a,0x8b6cdb0e141f9857d8c5f6cbfd2d4fe65f7b05a23ea1bdc5c2da2e1f49a0e51d"
  },
  {
    "at": 1788789071181,
    "totalUsd": 358.511623,
    "holders": 812,
    "realizedCumUsd": 108827.27370630718,
    "txHash": "0x599ead8e62dc9be1995ddb359688e461d1352be6e5f6b8749e8cbd2febbb9c2d,0xf057b8a9dfc3e27388bb6ce59b05d16c11fb5131525383ad7530ede1cfcb8408,0xd09399593cf81e4ec3d994e4d9c57d4091d4cf4dab0824474f4e256a5e05c8a7,0x7811b963a4753512d0113fc62d5f5bb759370fa308e8e1d1f44b33811ce7d76c,0x57ae63ef3b1798b59f988adecc15dfc9a5d0d6d61d2a65fc816dbcf4661916de"
  },
  {
    "at": 1788790328177,
    "totalUsd": 303.977466,
    "holders": 773,
    "realizedCumUsd": 109232.60605372285,
    "txHash": "0x075b4a8da453a67bbbde7e695eb35e61fbb6e65f1a1515dd7ee22400a82d4b34,0x089b5af56c58023f47521e83f39130c6d1bc82e953ab338c54a46f7344161aee,0xf95933d2f418b4586807ab73bac513f38fbd4cf2bf12d12e9629236b33cb49aa,0x9d4d5035939eea7a6fcf45d0df7bc12fdfcee6d90ae4f3eef6e33621f4b53916"
  },
  {
    "at": 1788793153919,
    "totalUsd": 357.110489,
    "holders": 802,
    "realizedCumUsd": 109708.65648672284,
    "txHash": "0xcbf3fe9f4091389d80d2a7b4a230acf5c82891f230d7934f35d96a1a319952e7,0xdbedabbf1db889eb9c6ee1d61ee6ea4849bb8ec4b14c698b191a85cf54d86b04,0xd39d78ba0c6aa330be181d880852cc4410db57aea688b7afe731a852258a04cd,0x431d8f4bae8bdcac40ceb66f2d81e85ad26cb72a386b6721d6f1282705740b91,0x7041b4b36d40e7df33aa5a140bdf76c3404ae07f88ad18bebd65a0eb1d61e9d1"
  },
  {
    "at": 1788794834907,
    "totalUsd": 377.414992,
    "holders": 804,
    "realizedCumUsd": 110211.89876358901,
    "txHash": "0x6802ba3964c6b52bda7d4328e8577a796a2695942b799adfecff8e29c7e29364,0x46b9519697391d7ab24fe3809ab4006cb58e9830a70e175e0946f6b2c64a73df,0xcb846e52ad27864bd291df5900490ce73ec9f42783984e395aba7bd9b841017d,0x90a1253e50a391ba3564d5b873e26a4417cad5cf98f4ebffd68fac2ddae73f58,0x8f8e807ea8648e685bc670900d384f3dd8aa76562f6e9ed3da85e7de5c093728"
  },
  {
    "at": 1788795803111,
    "totalUsd": 444.096505,
    "holders": 845,
    "realizedCumUsd": 110803.79802348975,
    "txHash": "0xc8709069269e936748a85041d7975ca1367f17bfcc510e51db8bf5306740f5e7,0x7159ca6c3b3fc12f859ee71fa117507c37f48f75f794a2021ecdbd05ae177fc7,0x3e957b7261e47df330fc140a48e04d467d3aae373dd85ad3d9ecf8a3db84021c,0x901749f99857fb0d578efacc0c261d4586d4022505490ee2ef000d3e73e708f2,0x2ae649a9117525c59dc284516707a2fb14068d83a11c27697e48e4622000d374"
  },
  {
    "at": 1788796763445,
    "totalUsd": 768.339509,
    "holders": 933,
    "realizedCumUsd": 111954.9035386394,
    "txHash": "0xb6469366b7f2309fe7909306f7f125f2f9215097ee5169f8afc8069a3bbb70c9,0x9e67abb842f7bd28e519adb06583fe5702f34f4d9606e0286933f32a1eb40412,0xf5200d025799e96823441b1987325d105e1cc2a07bbf9376b69929ba6dcf51f5,0x38d84ce850d53a6ae213094e8cebbb4215f20d97b95078d65567f8fcdba72146,0xae287d58211291dad7de58d82d9184f438555d98bf99af3389b50ac177d6dd5f"
  },
  {
    "at": 1788797722596,
    "totalUsd": 446.241218,
    "holders": 833,
    "realizedCumUsd": 112423.25793367648,
    "txHash": "0xd12f6ed38cb059d54abf7f0215914dbbe5ec03f1595081a2698c083bd50df5f9,0xcf5b5a185ed89a2cb46fc9c262a202271ba30dbbca4dbb7543dbdd9b9d022031,0x7f5d4c602e16570dbef7979592c822dd8cbfe616c6b2a3f8cb50b66d0e8886af,0x08c1a349709f9965930d88aea8b9d467c305ec867907713822e25a8e88475bd9,0xb875ed0b2561641eddf9b91f2bf4c0ee47a1f272f45f9a904485960406a51aef"
  },
  {
    "at": 1788798681691,
    "totalUsd": 351.404634,
    "holders": 771,
    "realizedCumUsd": 112891.94690976731,
    "txHash": "0xe3c76765a99a9a04dd71055fd1438e4d82b36bf321d49a7bbe73fa5182eb122c,0x22203317200c79a45f4738e471df0f8d47b3e54e51c0ea1da9a9b9c316f384af,0xbe3700fa3b00182d3329dd5769c7ab00ec562151887c263b90132cce46b1a11d,0x1e79f1fec43060f1b3bdaf5532a62e971d34df8800171da70e3acb0c80210bcb"
  },
  {
    "at": 1788799643408,
    "totalUsd": 300.796755,
    "holders": 739,
    "realizedCumUsd": 113293.05739776733,
    "txHash": "0xa75b688901ef1e9a1a1319b13c767bcb4203b0ac23672155dd0902a82a2dd697,0xad1dca5d64a69696370c9f33b86e24ca4b033ae11713ac1048933951388db19c,0x93f325b0a298b43e41541f6b5ff88ef93cc30986da3ae16a80ae8cd8b5b659e5,0x52a65fa6327c5cac8801e9629cf8b4894e429f5df34fd50bb1a946f578a1a813"
  },
  {
    "at": 1788800783483,
    "totalUsd": 353.295634,
    "holders": 779,
    "realizedCumUsd": 113764.08691274229,
    "txHash": "0x39a512d3765168031be58973353f1aa54404edb4725361c14ea0d9e5a88f5ff8,0x3541cbd33c83af9f1c29128774918e90bfdb37925f02e819df6b6bd8a51a4db4,0x4aaf6c098f5789041984cce14d6d051a30e04b927d67605d82c5542ed5853740,0xac6c69b3b08fb15517bae09fab6e127f5f68a82e382c95ab0f084a31b59b774e"
  },
  {
    "at": 1788801745634,
    "totalUsd": 329.394236,
    "holders": 765,
    "realizedCumUsd": 114203.28435472584,
    "txHash": "0x182359d981bb0a1ba95c7658de538e376134ea921ad9f46577f12a3f1a3ba983,0xc9adaa515cae625829681dee4b5b6c83565df9a228d5ce73a3d7d9575fc8d955,0x38e853382a1a0867b766942de445d1fabe89b4e97786f95323b95bb9a337a4ff,0x542b0c8a7b0f6e094d3cb5e1276602e91dc1fb262d8fdee507c9d3cc837f1707"
  },
  {
    "at": 1788804391687,
    "totalUsd": 376.235837,
    "holders": 787,
    "realizedCumUsd": 114704.93862028646,
    "txHash": "0xbf6637af9c991bc33e2fd012bf0c8ce2d965ff78ea8c8f7e71cd22b988043ae7,0x603de4ef565b5b8f96fef8e6490b486319e1236667f9795cd63b5fe0af249065,0x5c52f27aa3ea540f8c00b0650421bf0990d1cb7b5a29aa1169f38dd917eede6e,0x8b44dddc5dbefd48730463708b59debfbd3f73bafc33cd5c9f968dd402accc31"
  },
  {
    "at": 1788805980850,
    "totalUsd": 307.322683,
    "holders": 745,
    "realizedCumUsd": 115114.73472361562,
    "txHash": "0x3723ea5ed471d7a4230552d35a7d1fc909714b5693b934e75953395a9fcc83cf,0x8f847fbcd778fb491bf8130a7ba9c23f6107d77160caf5826f00c33856daa3b5,0x98edf716a528b484abcf7fb0f57e665a2a7527c604852be1e5c5f3a481fe7e71,0xc3321739ee422c785541c26037206d10a1b3557eef85742268acbe3678527c10"
  },
  {
    "at": 1788806941989,
    "totalUsd": 299.136405,
    "holders": 741,
    "realizedCumUsd": 115513.56268400312,
    "txHash": "0x62d909cdece4ba666c46affa787db1a4283af4e19697ebbe49a723f896a5f417,0xe19f10c9a2ec5307bed2ceab70b58912e8f9932f6b5ad28eca61da60e32eacd9,0x4d7df613dbd222087370dba1c24509134340326497d86f33770fc18ababac1d7,0xf143b7dff79ade38dd8482150ff6cbfe4c70a0a2c81f04ab084c15de323fb060"
  },
  {
    "at": 1788807912244,
    "totalUsd": 604.177908,
    "holders": 890,
    "realizedCumUsd": 116318.89418508581,
    "txHash": "0x2f582e4f5d8a1e207ea7a2f53100ac7954cc3f636a57dcd4329df18560f65152,0x9a04cb11155d104c703618122467e97c3db9e199cfdf958516cfbe8d7079be49,0xa2c09d0782ed6faadcb46e36d48717fa03cf18366da54a0960f86066710dda6c,0xb9920996b716664687129e7aa26f72870991f6ba60614db16a50e7e2849a815a,0x69fbecaf56ac802e7baab73cc87b4c35fd1904cd3fd392a5397c5be8b0510513"
  },
  {
    "at": 1788808870219,
    "totalUsd": 498.896506,
    "holders": 856,
    "realizedCumUsd": 116984.08479977591,
    "txHash": "0x43a4a7a4b6e094670e9ba6461bcdec201a2106868641b02e34a8f7b3d4275371,0x3d07104f42ab41329028d022daa359d6a3b55476e303d38b8af4028a2ab60ae9,0x88b3c64882855824fb516fae972d994a805c431375dd2611d8b2474c4ce83b7e,0x8c6eda34d4d1599643d7b7ae1fdd12149c35af2e8d6e46c0e2edc145b8b2edd4,0x7b9a2c3f719786053e19ed2bc16303d257f5f47501cda81fd49140840026ddc0"
  },
  {
    "at": 1788809834916,
    "totalUsd": 477.820844,
    "holders": 851,
    "realizedCumUsd": 117621.24122064875,
    "txHash": "0x11ff524b78f7695fe7f9297f3d31890d7918a6ee1ec122bb4192a9498a316e81,0x72548676e8874013e618b029d7da89f69b515bee17b40f2d5ab119730427b04c,0x8dd7ddf699bd31d4d811fa873075298b3732634aef17154cdd13837fbae8ba38,0xfdee3fac605d58c73a4a0de26d8a421565891a613c46d0dac2cb7c1f15e1693e,0x99c4de8c08dd401e260e7407a53c4d54b6bd9a48e08da1cbe79b2d507b008acf"
  }
]
```

## Appendix E — Methodology notes

- Chain reads via private Robinhood RPC (`ROBINHOOD_RPC_URL`; URL not published)
- Selectors confirmed in vault bytecode at pin
- State pinned to block `57099428` / `0x80e6ce7ba61459ef97604844b2c75a7123a183531275b0bb53fa1d5b2366f15c` / `2026-09-07T19:54:31Z`
- USDG **6** decimals; AA **18**
- No real keys used; no transactions signed or broadcast

## Appendix F — Target integrity manifest (embedded)

```json
{
  "requested_chain_id": 4663,
  "requested_address": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
  "observed_chain_id": 4663,
  "observed_address": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
  "metadata": {
    "name": "Arbitrage Ape",
    "symbol": "AA",
    "decimals": 18,
    "total_supply": "1000000000000000000000000000",
    "unresolved": []
  },
  "pins": [
    {
      "label": "state_head",
      "block_number": 57099428,
      "block_hash": "0x80e6ce7ba61459ef97604844b2c75a7123a183531275b0bb53fa1d5b2366f15c",
      "utc": "2026-09-07T19:54:31Z"
    }
  ],
  "runtime": {
    "verified_status": "PonsV2LauncherToken verified; vault unverified"
  },
  "scope_addresses": [
    {
      "address": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
      "role": "token",
      "chain_id": 4663,
      "provenance": "user",
      "runtime_status": "contract"
    },
    {
      "address": "0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98",
      "role": "vault",
      "chain_id": 4663,
      "provenance": "project_api_fund",
      "runtime_status": "contract_unverified"
    },
    {
      "address": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
      "role": "USDG",
      "chain_id": 4663,
      "provenance": "project",
      "runtime_status": "erc20"
    },
    {
      "address": "0x12b46b7746af4a902fd55198f09ecfd8c4d42956",
      "role": "vault_owner",
      "chain_id": 4663,
      "provenance": "owner()_eth_call",
      "runtime_status": "eoa"
    },
    {
      "address": "0xdef933cfaeb2a2af516df1eaa2101b00b7f77af6",
      "role": "keeper_observed",
      "chain_id": 4663,
      "provenance": "distribution_tx_from",
      "runtime_status": "eoa"
    }
  ],
  "report_source_id": "arbitrage-ape-dd-v4-refresh",
  "no_real_signing": true,
  "no_broadcast": true
}
```
