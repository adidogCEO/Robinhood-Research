# Arbitrage Ape ($AA) — Due Diligence Report

**Generated:** 2026-09-05 ~23:15 UTC (merged v3)  
**Chain:** Robinhood Chain (`4663`) — RPC-verified  
**Exact token:** `0xe2dae1c072b9f66bed999873b94798b2152af7e1`  
**Scope:** Product mechanics, custody/permission map, risks, onchain-verified payouts, skill-grade surface scoring  
**Format:** Single self-contained Markdown (status + full payout history + target pin embedded)  
**Skill:** [evm-token-due-diligence](sand-workflow:evm-token-due-diligence)

### State pin
| Field | Value |
|---|---|
| Label | `state_head` |
| Block | `55498712` |
| Block hash | `0x905feebb6faf85315a8f2697e121f538dcc28d4e58fd8e102d59dc3c4df65823` |
| UTC | `2026-09-05T23:03:40Z` |

---

## Executive summary

Arbitrage Ape is a **live, programmatic** market-making / dislocation desk on Robinhood Chain—not a pure LARP. A funded vault holds inventory and cash; an automated keeper calls `exec` and `distribute` on a cadence; holder payouts are real **USDG** transfers from the vault.

It is also **maximally centralized**: the vault is **unverified**, the **owner can withdraw any asset with no timelock**, and the operator is effectively anonymous. Treat `$AA` as a speculative claim on desk profit flow, not trustless custody.

| Question | Answer |
|---|---|
| Real onchain capital + automated execution? | **Yes** |
| Holder payments real USDG? | **Yes** (RPC-verified samples + live API) |
| Empty “waiting for keeper” UI = inactive? | **No** — SSR placeholders; `/api/*` and chain are live |
| Trustless / rug-resistant vault? | **NO-GO** — `withdraw` present; no timelock |

### Surface scores (pinned state)

| Surface | Rating |
|---|---|
| Token controls | Moderate concern (Pons v2 token verified; vault is control plane) |
| Canonical LP-principal custody | High concern (vault/`exec` funded LP; owner can withdraw) |
| Side-pool removal risk | Unknown / elevated |
| Sellability and exit depth | Unknown (coverage — no holder-sized quotes this run) |
| Current concentration | Elevated (vault ~10.7% `$AA`) |
| Historical launch integrity | Partial (Pons v2 confirmed; SIZE-cohort not completed) |
| Admin / treasury / reward custody | **Critical** |
| Reward accounting and liveness | Strong (operational) |
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
| X | https://x.com/ArbitrageApe (created ~2026-09-04) |
| Vault / fund | `0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98` |
| `$AA` fee token | `0xe2dae1c072b9f66bed999873b94798b2152af7e1` — **Arbitrage Ape** / `AA`, 18 decimals, 1B supply; Blockscout name `PonsV2LauncherToken` (verified) |
| USDG | `0x5fc5360d0400a0fd4f2af552add042d716f1d168` — **6 decimals** |
| Keeper (EOA) | `0xdef933cfaeb2a2af516df1eaa2101b00b7f77af6` |
| Owner | `0x12b46b7746af4a902fd55198f09ecfd8c4d42956` (`owner()` eth_call at pin) |
| Explorers | [Blockscout](https://robinhoodchain.blockscout.com/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98) · [Robinscan](https://robinscan.io/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98) |
| Live APIs | `/api/status`, `/api/distributions`, `/api/trades`, `/api/positions`, … |

**Live snapshot (API at report generation; pin `55498712`):**

| Metric | Value |
|---|---|
| Mode | `live` |
| Vault ETH | ~1.2677506289738811 ETH |
| Vault USDG | ~**$18075.664799** |
| Vault `$AA` (onchain) | ~**107.15M** (~10.7% supply) |
| Realized profit | ~$49116.86 |
| Distributed (API) | ~**$48808.79** |
| Pending owed | ~$308.06 |
| Fee flow claimed (lifetime) | ~**$159395.35** |
| LP principal PnL (API) | **~$-36962.94** |
| Instruments / eligible | ~194 / ~89 |
| Pools tracked v3/v4 | 373 / 7652 |
| `$AA` mark (API) | ~$0.0017902589788860127 · mcap ~$1790259 |

## 2. How it works

### 2.1 Market premise

Tokenized equities on Robinhood Chain trade across fragmented Uniswap v3/v4 (and related) venues. Thin depth means a single aggressive order can push a pool **far above** the primary-market share price. That dislocation is inventory-constrained profit: whoever already holds the asset can sell into it.

The desk’s design: **be standing inventory** where dislocations happen, and **sell under strict profit rules** when they do.

### 2.2 Capital formation

1. `$AA` launches on **Pons v2**; the vault is the **creator-fee recipient**.
2. Creator fees accrue in USDG to Pons escrow; the keeper **claims into the vault** (API: ~$149k claimed lifetime).
3. Fee cash buys thin / pump-prone stock tokens **at or under reference** (rotation, clip caps, max acquisition premium), building inventory for later sells.
4. Optional **LP bands** around premium pools; collected fees also enter the holder profit pot.

### 2.3 Pricing and dislocation filter

- Reference \(\hat{P}\) from primary quotes (Chainlink fallback); ~15s refresh.
- Deviation \(\delta = (P_{pool} - \hat{P}) / \hat{P}\).
- Typical actionable threshold **≥ +25%** (widened when NYSE session closed).
- **Fillability probe**: simulated quote of a fixed notional through the pool’s quoter—mirage prices that cannot fill are discarded.
- Deep instruments (tiny impact at $10k) are excluded; thin eligible set is re-surveyed on a schedule.

### 2.4 Execution

- When a held name prints a real dislocation, size the max sell whose **effective** price clears a floor vs reference and cost basis.
- Tranche exits (fraction per cooldown) rather than slamming the whole book.
- Onchain path: keeper → vault **`exec(address,uint256,bytes)`** (`0x0565bb67`), interacting with allowlisted venues (e.g. Uniswap v4 PoolManager observed in logs).

### 2.5 Profit accounting and holder payouts

- Realized profit \(\Pi\) accumulates from sells (proceeds − average cost of size sold) and fee collections booked to the pot.
- Owed to holders: \(O = \Pi - D\) (never resets).
- About every **15 minutes**, once \(O \ge\) ~**$300**, pay `min(O, cash)` in **USDG** pro-rata to eligible `$AA` holders (AMM/desk addresses excluded).
- Large holder sets are split across **multiple distribution txs** per round (~200 recipients per tx observed).
- Onchain path: keeper → vault **`distribute(address,address[],uint256[])`** (`0x15270ace`).

### 2.6 Why the website can look “dead”

SSR/marketing HTML often shows “Waiting for the keeper…”, “Last payout: none yet”, empty books. **Live JSON APIs are populated.** Prefer `/api/status`, `/api/distributions`, and explorer txs over the empty chrome.

---

## 3. Custody and permission map (bytecode-level)

Vault runtime bytecode ≈ **4803 bytes**. **Source not verified** on explorers (`isVerified: false`). Selectors resolved via public signature databases:

### 3.1 Roles

| Role | Address | Notes |
|---|---|---|
| `owner()` | `0x12b46b77…42956` | Ownable2Step; `pendingOwner()` currently zero |
| `keeper()` | `0xdef933cf…7af6` | Matches live `exec` / `distribute` caller |

### 3.2 Owner / admin surface

| Function | Selector | Risk implication |
|---|---|---|
| `owner()` | `0x8da5cb5b` | Single privileged admin |
| `transferOwnership(address)` | `0xf2fde38b` | Ownership can move |
| `acceptOwnership()` | `0x79ba5097` | Two-step accept |
| `renounceOwnership()` | `0x715018a6` | Can renounce (unlikely while operating) |
| `withdraw(address,address,uint256)` | `0xd9caed12` | **Owner can pull any ERC-20/asset to any recipient — no timelock (docs + ABI)** |
| `setKeeper(address)` | `0x748747e6` | Swap the automation agent |
| `setTarget(address,bool)` | `0x80ffe77f` | Allowlist execution targets |
| `setSpender(address,bool)` | `0x6be3f3e0` | Allowlist spenders / approvals |
| `setDailyCap(address,uint256)` | `0xda09a331` | Cap distribution/outflow per asset |
| `approveToken(address,address,uint256)` | `0xda3e3397` | Owner-gated approvals |

### 3.3 Keeper / automation surface

| Function | Selector | Observed use |
|---|---|---|
| `exec(address,uint256,bytes)` | `0x0565bb67` | Trading, LP, fee claims — every ~1–5 min |
| `distribute(address,address[],uint256[])` | `0x15270ace` | Holder USDG payouts — ~15–18 min batches |
| `isTarget(address)` | `0xaa642274` | View allowlist |
| `isSpender(address)` | `0x9a206ece` | View spender allowlist |

Docs claim `exec` reverts if target not allowlisted (`TargetNotAllowed`). Approvals are intended to be spender-gated the same way.

### 3.4 Distribution rate limits (onchain views)

| Asset | `dailyCap` | Interpretation |
|---|---|---|
| USDG | `50000000000` raw | **50,000 USDG / rolling day** (6 decimals) at check time |
| AA | `0` | No AA distribution cap configured (or unused) |
| ETH (address zero) | `0` | No ETH dist cap |

Related views/errors in bytecode: `windowStart(address)`, `CapExceeded`, `InsufficientBalance`, `ReentrancyGuardReentrantCall`.

**Important:** Daily caps constrain the **keeper distribution path**. They do **not** neutralize **`owner.withdraw`**, which remains the dominant rug vector.

### 3.5 Invariants (claimed vs reality)

| Claimed invariant | Assessment |
|---|---|
| Balances live at one fund address | **Holds in practice** — inventory/cash at vault |
| Agent wallet holds only gas | **Consistent** — keeper calls vault; nonce ~2820 |
| Agent replaceable | **Yes** — `setKeeper` |
| Distribution rate-limited | **Partial** — USDG daily cap on; owner withdraw unconstrained |
| Owner can withdraw anytime | **True** — documented and present in ABI |

---

## 4. Risks

### Critical

1. **Instant owner drain** — `withdraw` with no timelock. Holding `$AA` for yield does not protect vault capital from the deployer.
2. **Unverified custody** — No public verified source; no third-party audit found.
3. **Anonymous / day-old social presence** — Hard to attribute accountability.

### High

4. **Single keeper** — Automation can stop, be key-compromised, or be replaced maliciously by owner.
5. **Economic fragility** — API already shows large **negative LP principal PnL** while distributions continue. Desk can lose inventory/LP capital even as the “realized pot” pays holders.
6. **Circular / reflexive revenue** — Fee token self-MM, high-fee pools, and creator-fee → buy inventory → sell spikes can be fragile or partially circular.

### Product / market

7. Edge depends on thin stock-token pools and closed-market dislocations; competition or deeper liquidity shrinks it.
8. Oracle / reference failure when NYSE closed or feeds lag.
9. `$AA` price ≠ vault NAV; it is a speculative claim on future distributions.

### UX / perception

10. Empty SSR dashboard vs live APIs makes the product easy to dismiss as theater—or easy to overtrust without reading permissions.

---

## 5. Payout rounds — desk-wide totals (refreshed)

| Metric | Value |
|---|---|
| Distribution rounds | **164** |
| Onchain payout transaction hashes | **561** unique |
| Rounds using multiple txs | **115** |
| Sum of API round totals | **$48,633.68** |
| Desk `distributedUsd` | **$48,808.79** |
| First round | 2026-09-04 13:19 UTC · $0.8666 · 33 holders |
| Latest round | 2026-09-05 23:02 UTC · $702.25851 · 1041 holders · 6 txs |
| Largest round | 2026-09-05 20:39 UTC · **$2,779.56** · 1346 holders |
| Cadence | ~15–18 minutes |

### Size distribution (by round `totalUsd`)

| Band | Rounds | Sum USD |
|---|---|---|
| < $1 | 11 | $8.22 |
| $1–10 | 24 | $70.27 |
| $10–100 | 39 | $2,106.71 |
| $100–500 | 55 | $16,760.78 |
| $500–1,000 | 26 | $16,708.74 |
| ≥ $1,000 | 9 | $12,978.97 |

Full machine-readable feed (also embedded in Appendix D): https://www.arbitrageape.app/api/distributions

---

## 6. Onchain-verified payout samples

Method: Robinhood public RPC `eth_getTransactionReceipt` — sum USDG `Transfer` logs **from the vault** (6 decimals). Caller in all cases: **keeper** → **vault**, selector **`distribute` (`0x15270ace`)**, status success.

### A) Earliest round — exact match

| Field | Value |
|---|---|
| Time | 2026-09-04 13:19 UTC |
| API total | **$0.8666** |
| Holders | 33 |
| Tx count | 1 |
| Tx | `0x567e40ea130d06d74192387b2fe240c659dc2b70f50c51ad39b8ccdcf14a2f51` |
| Onchain USDG from vault | **$0.866600** |
| Recipient transfers | **33** |
| Explorer | https://robinhoodchain.blockscout.com/tx/0x567e40ea130d06d74192387b2fe240c659dc2b70f50c51ad39b8ccdcf14a2f51 |

### B) Largest round — multi-tx (first 2 of 7 verified)

| Field | Value |
|---|---|
| Time | 2026-09-05 14:06 UTC |
| API total | **$1,514.41** |
| Holders | 1,283 |
| Tx count | 7 |

| Tx | USDG from vault | Recipients |
|---|---|---|
| `0xfb07d1583312058f9469cde4d757b4aa0898729d6db657bd339181bc193cd581` | **$1,278.75** | 200 |
| `0xe3d6bc5a343f6f1d8df7c0135a32b44f0ee9681a6953ad02705b83e87b1dc815` | **$146.18** | 200 |
| **Subtotal (2/7)** | **$1,424.93** | — |

Remainder (~$89) sits on the other five txs in the same round—consistent with batching.

### C) Latest round — multi-tx (first 2 of 5 verified)

| Field | Value |
|---|---|
| Time | 2026-09-05 16:50 UTC |
| API total | **$405.69** |
| Holders | 968 |
| Tx count | 5 |

| Tx | USDG from vault | Recipients |
|---|---|---|
| `0x6ddbe65a384f0bbca8b8fd65a71b0011129839de5c2b306f8e4dace8be88924c` | **$342.02** | 200 |
| `0xbee3b560409a0a71af6e7d4304ea0af50bdab068a5b3d3bfa82f88d5c91cd85a` | **$41.12** | 200 |
| **Subtotal (2/5)** | **$383.13** | — |

### Pattern

- Small rounds: **1 tx**, recipient count ≈ holder count, USDG sum **matches API to the cent**.
- Large rounds: **several sequential keeper txs**, often **~200 transfers each**, totaling the round.
- Trading uses **`exec`**; payouts use **`distribute`**—different code paths, both automated.

---

## 7. Holder distribution mechanics

System-level recipient behavior from docs and observed `distribute` txs:

- Payouts are **pro-rata `$AA` balance** at snapshot (docs: local Transfer history, spot-checked vs chain).
- Eligible supply excludes AMM reserves, protocol machinery, and desk addresses.
- Sub-dust allocations stay in the pot.
- Observed onchain: multi-recipient USDG fans from the vault in a single `distribute` call; larger rounds chunk addresses across txs.
- Holder counts grew from **~33** (first round) to **~900–1,300** on recent large rounds.

---

> **Note:** Samples A–C above were RPC-verified at the original ~17:10 UTC generation. They remain valid historical evidence. Desk totals above are **refreshed** to the pinned/live API state.

## 7. Holder distribution mechanics

System-level recipient behavior from docs and observed `distribute` txs:

- Payouts are **pro-rata `$AA` balance** at snapshot (docs: local Transfer history, spot-checked vs chain).
- Eligible supply excludes AMM reserves, protocol machinery, and desk addresses.
- Sub-dust allocations stay in the pot.
- Observed onchain: multi-recipient USDG fans from the vault in a single `distribute` call; larger rounds chunk addresses across txs.
- Holder counts grew from **~33** (first round) to **~900–1,300** on recent large rounds.

---

## 8. Concentration (holders page, refreshed)

Blockscout `holders_count` ≈ **1700** at fetch time. Top of first holders page:

| Address | % supply | Note |
|---|---|---|
| `0x0feB08fcF34F5c0e270261bC79BaFC0F8eb87f98` | 10.7154% | vault |
| `0x267444D099b10fB5Ed7c3Cc7B7c767AdcA574952` | 8.1633% |  |
| `0x8366a39CC670B4001A1121B8F6A443A643e40951` | 4.3437% | Uniswap v4 PoolManager (LP inventory) |
| `0x47bfaF27eD550FfaA24a593E772A5947AF9f6523` | 3.0013% |  |
| `0x05607d87494C3f9D6e43Be2C2E5Ed23A95574548` | 2.636% |  |
| `0xB3F4bE999FD39f4a56F5a9713DcAed3Edb310a14` | 2.632% |  |
| `0xE46a27A4e371d10F5A8155f44da25eb5F91fA835` | 2.3488% |  |
| `0xa5239426423e2600C1DC35A8fce400929C547e4d` | 1.4299% |  |
| `0x2032f86ef7f40e91cA1B1AF304A17d9Ad3881a90` | 1.3048% |  |
| `0x4d8eDF53399736052850646b43E3329c66BB49DC` | 1.2176% |  |
| `0x77dc0CA15471d42Bd326279526e8d6D7EE10E74a` | 1.1999% |  |
| `0x0990A6A05bFF0c441D99b7301bB2b07816Eab496` | 1.1065% |  |
| `0x4a1281eb29Bc0061123dbbe9DA699e3F255EbEE7` | 1.0993% |  |
| `0x696d1265C8Fc4F14797aBEBFAe3C43EBFA9D8e28` | 1.0016% |  |
| `0x3d957C9Ea42C8743b462abD451E3c93F249D4ace` | 0.9965% |  |

Vault alone ≈ **10.7%** of supply. Uniswap v4 PoolManager balance is **pool inventory**, not a retail holder.

---

## 9. Finding-to-evidence ledger (new)

| ID | Proposition | Evidence | Confidence | Stale if |
|---|---|---|---|---|
| F1 | Chain is `4663` | `eth_chainId` in pin | High | other chain |
| F2 | Exact `$AA` is `0xe2dae1c072b9f66bed999873b94798b2152af7e1` | token meta + holders | High | — |
| F3 | Vault `owner()` is `0x12b46b7746af4a902fd55198f09ecfd8c4d42956` | eth_call `0x8da5cb5b` at pin | High | ownership transfer |
| F4 | `withdraw(address,address,uint256)` in vault runtime | selector `0xd9caed12` in `vault.runtime.hex` | High | new deploy |
| F5 | `exec` / `setKeeper` / `keeper` / `0x15270ace` present | runtime selector scan | High | new deploy |
| F6 | Vault unverified | Blockscout `is_verified: false` | High | verification |
| F7 | Live distributions accruing | `/api/status` + 164 rounds | High | API change |
| F8 | Early distribute succeeded | tx `0x567e40ea…2f51` keeper→vault | High | — |
| F9 | Vault ~10.7% `$AA` | holders page + balance call | High | transfers |
| F10 | `$AA` is Pons v2 launcher token | Blockscout contract name/creator | High | — |
| F11 | Holder-sized exit depth OK | **not obtained** | — | — |
| F12 | Launch cohort clean (SIZE-pattern) | **not investigated** | — | — |
| F13 | Owner never withdrew treasury | **not proven** | — | — |

---

## 10. Bottom line

**Real desk. Real keeper. Real USDG payouts.**  
**Not trustless. Soft-rug capable by design (owner withdraw).**  
**NO-GO under a rug-resistance requirement**; acceptable only as **operator-dependent speculative yield** on `$AA`, with custody and LP/inventory risk the distribution feed alone will not show.

### Coverage limits (not passes)
1. Pinned holder-sized `$AA` sell quotes  
2. SIZE-pattern launch-cohort accounting  
3. Receipt-level fee→inventory→distribute conservation + owner-withdraw history  
4. Exhaustive side-pool LP inventory beyond API bands  
5. `pendingOwner()` at pin (RPC 429 on one call)

---

## Appendix A — Key links

- Desk: https://www.arbitrageape.app/
- Docs: https://www.arbitrageape.app/docs
- Status API: https://www.arbitrageape.app/api/status
- Distributions API: https://www.arbitrageape.app/api/distributions
- Vault (Blockscout): https://robinhoodchain.blockscout.com/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98
- Vault (Robinscan): https://robinscan.io/address/0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98
- `$AA`: https://robinhoodchain.blockscout.com/token/0xe2dae1c072b9f66bed999873b94798b2152af7e1
- Public prior copy: https://github.com/adidogCEO/Robinhood-Research/blob/main/Arbitrage-Ape.md

## Appendix B — Live desk status (embedded)

Snapshot of `GET https://www.arbitrageape.app/api/status` at this report generation (refreshed). Formerly a separate `status.json`.

```json
{
  "mode": "live",
  "feeToken": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
  "fund": "0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98",
  "ethBalance": "1.2677506289738811",
  "ethUsd": 2484.62495218,
  "ethFloatTargetUsd": 3000,
  "usdgBalance": 18075.664799,
  "realizedProfitUsd": 49116.8582435298,
  "operatorPnlUsd": -53.58,
  "lpPrincipalPnlUsd": -36962.94028217039,
  "distributedUsd": 48808.79344900001,
  "minDistributionUsd": 300,
  "pendingProfitUsd": 308.06479452978965,
  "lastDistributionAt": 1788649332671,
  "nextDistributionAt": 1788650232671,
  "lastScanAt": 1788650141825,
  "lastSurveyAt": 1788574309360,
  "referenceAt": 1788650141170,
  "marketOpen": false,
  "thresholdBps": 3500,
  "pools": {
    "v3": 373,
    "v4": 7652
  },
  "instruments": 194,
  "eligible": 89,
  "feed": {
    "live": true,
    "lastEventAt": 1788650141726,
    "events": 45279,
    "since": 1788648072668
  },
  "backfill": {
    "head": "55505623",
    "v3": "55505624",
    "v4": "55505624",
    "caughtUp": true,
    "at": 1788650117062
  },
  "feeFlow": {
    "escrowUsd": 0,
    "claimedUsd": 159395.3490740001,
    "lastClaimAt": 1788648609645,
    "phase": 2,
    "recipientIsVault": true,
    "at": 1788650106324
  },
  "token": {
    "priceUsd": 0.0017902589788860127,
    "mcapUsd": 1790258.9788860127,
    "supply": 1000000000,
    "at": 1788650143393
  },
  "lp": {
    "enabled": true,
    "bands": [
      {
        "symbol": "AI",
        "band": "AI",
        "pool": "0x7aebd80541bfaaf23dbb6e99ce13d4d31c1a84c91414f971eadbff7db5f85995",
        "protocol": "v4",
        "capitalUsd": 2000,
        "mode": "two-sided",
        "fee": 2300,
        "stockIs0": true,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 2500,
        "hooked": false,
        "position": {
          "tokenId": "1917064",
          "lowerUsd": 0.2545573955240883,
          "upperUsd": 0.39861813749719627,
          "depositUsd": 882.85024,
          "depositQty": 2872.0961202523567,
          "depositStockCostUsd": 912.2449209537732,
          "depositQuoteQty": 0,
          "stockQty": 5232.044397843999,
          "cashUsd": 194.84987259319865,
          "quoteQty": 194.84987259319865,
          "valueUsd": 1596.096097734623,
          "feesUnclaimedUsd": 0.8547193526085889,
          "feesCollectedUsd": 63.735693612517544,
          "inRange": true,
          "openedAt": 1788622632173,
          "txHash": "0x785340acf6f604fd5472bb0c0de5682c70cc97df959199e0cecbaf2ecb100e84"
        },
        "spotUsd": 0.2678200180638461,
        "refUsd": 0.2678200180638461,
        "feesCollectedUsd": 188.9202586335725,
        "lastAction": null,
        "at": 1788650145449
      },
      {
        "symbol": "AA",
        "band": "AA",
        "pool": "0xe2156fc6454acef70a7763cf0f90e207c182880388ab3d381b84dc5d2e91b1bf",
        "protocol": "v4",
        "capitalUsd": 30000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": {
          "tokenId": "1959518",
          "lowerUsd": 0.0010413326425273775,
          "upperUsd": 0.0024362471030970516,
          "depositUsd": 13043.90594,
          "depositQty": 8402565.8619081,
          "depositStockCostUsd": 0,
          "depositQuoteQty": 0,
          "stockQty": 5841316.2373495735,
          "cashUsd": 17357.721915912505,
          "quoteQty": 17357.721915912505,
          "valueUsd": 13043.90594,
          "feesUnclaimedUsd": 0,
          "feesCollectedUsd": 792.7190459130621,
          "inRange": true,
          "openedAt": 1788648388358,
          "txHash": "0xac1ded7e070e784764cf3c74d79083afcfdc6b122199f804e5da063904acd6d1"
        },
        "spotUsd": 0.0017897823772803907,
        "refUsd": 0.0017900206780832018,
        "feesCollectedUsd": 26838.855143109387,
        "lastAction": null,
        "at": 1788650146317
      },
      {
        "symbol": "PONS",
        "band": "PONS",
        "pool": "0x5cf9e7a7c416d1a1673a013c1bb311a1cf57be2944616949c2dd15e4624a3bff",
        "protocol": "v4",
        "capitalUsd": 10000,
        "mode": "two-sided",
        "fee": 7000,
        "stockIs0": true,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": {
          "tokenId": "1882106",
          "lowerUsd": 0.5285776927669701,
          "upperUsd": 1.2000863642810538,
          "depositUsd": 4466.79144,
          "depositQty": 5680.9038869999995,
          "depositStockCostUsd": 4475.322743821929,
          "depositQuoteQty": 0,
          "stockQty": 3585.1734212662523,
          "cashUsd": 6254.872126948857,
          "quoteQty": 6254.872126948857,
          "valueUsd": 9554.640282183835,
          "feesUnclaimedUsd": 0,
          "feesCollectedUsd": 107.10558963084192,
          "inRange": true,
          "openedAt": 1788596219535,
          "txHash": "0x99d4e6e99f2ecf46237264f96b6bfd611928aaafa870afd3b05b2d6e185510d8"
        },
        "spotUsd": 0.9161727428135038,
        "refUsd": 0.9203928980566656,
        "feesCollectedUsd": 107.10558963084192,
        "lastAction": null,
        "at": 1788650146966
      },
      {
        "symbol": "GRASS",
        "band": "GRASS",
        "pool": "0xf726e10ca81d42d05997c23df1d3f982c831b355d3a0723f830fdf433d893b36",
        "protocol": "v4",
        "capitalUsd": 6000,
        "mode": "two-sided",
        "fee": 39999,
        "stockIs0": true,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": {
          "tokenId": "1882067",
          "lowerUsd": 0.005476237543242361,
          "upperUsd": 0.012684443079892324,
          "depositUsd": 2700,
          "depositQty": 317313.8830412519,
          "depositStockCostUsd": 2953.851985237358,
          "depositQuoteQty": 0,
          "stockQty": 58156.4215376573,
          "cashUsd": 5261.3145270533205,
          "quoteQty": 5261.3145270533205,
          "valueUsd": 5936.222432659597,
          "feesUnclaimedUsd": 6.462053746452456,
          "feesCollectedUsd": 2993.6100377610114,
          "inRange": true,
          "openedAt": 1788596195489,
          "txHash": "0xa9fa472ce3f87b4a1f53d7de8b3b62787cd1a4d9867410df310fced04270cb45"
        },
        "spotUsd": 0.011673936766853223,
        "refUsd": 0.011605045285829044,
        "feesCollectedUsd": 2993.6100377610114,
        "lastAction": null,
        "at": 1788650147637
      },
      {
        "symbol": "DUST",
        "band": "DUST",
        "pool": "0xe4a994f090844198d1fe5da6386195231a62514e430f129d1a439b49403aea2e",
        "protocol": "v4",
        "capitalUsd": 10000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": {
          "tokenId": "1949642",
          "lowerUsd": 0.0005715126152419777,
          "upperUsd": 0.0014056309559150014,
          "depositUsd": 4498.147507,
          "depositQty": 4983147.7809687,
          "depositStockCostUsd": 5224.234850714932,
          "depositQuoteQty": 0,
          "stockQty": 9018831.943046203,
          "cashUsd": 1382.7253614297433,
          "quoteQty": 1382.7253614297433,
          "valueUsd": 7344.556549023837,
          "feesUnclaimedUsd": 36.452428602115546,
          "feesCollectedUsd": 1116.3143342287615,
          "inRange": true,
          "openedAt": 1788641157777,
          "txHash": "0x9c83ab3803ee4c1aa8cdcd8aefbc87d982435971eb09f06af8b48f25357a0bae"
        },
        "spotUsd": 0.0006639419093029526,
        "refUsd": 0.0006610424969932885,
        "feesCollectedUsd": 6140.130548411153,
        "lastAction": null,
        "at": 1788650148500
      },
      {
        "symbol": "LDX",
        "band": "LDX",
        "pool": "0x2ac316cc5ae07e6d0debc06c8ea59c44cc4e01191ecd392f4e47e6a9135a5c12",
        "protocol": "v4",
        "capitalUsd": 10000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": {
          "tokenId": "1953933",
          "lowerUsd": 0.0012404727171854628,
          "upperUsd": 0.0028304940807556277,
          "depositUsd": 4313.193108,
          "depositQty": 2327208.6817683,
          "depositStockCostUsd": 4509.143867788259,
          "depositQuoteQty": 0,
          "stockQty": 4645412.822159398,
          "cashUsd": 659.4926419180676,
          "quoteQty": 659.4926419180676,
          "valueUsd": 6753.807167446863,
          "feesUnclaimedUsd": 30.680229718896385,
          "feesCollectedUsd": 723.079400417152,
          "inRange": true,
          "openedAt": 1788644917927,
          "txHash": "0xc8a3c5b4d836ed1e40c1d119780697614861d73cc14e269bdacd72cdde863387"
        },
        "spotUsd": 0.0013283884329809957,
        "refUsd": 0.001311899449809475,
        "feesCollectedUsd": 766.3718104171519,
        "lastAction": null,
        "at": 1788650149556
      },
      {
        "symbol": "CATSTRO",
        "band": "CATSTRO",
        "pool": "0x0f04fcb8225d70137b16e3145255a93a7565f2c9e0c8307958ef71220a44d385",
        "protocol": "v4",
        "capitalUsd": 2000,
        "mode": "under",
        "fee": 70000,
        "stockIs0": true,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": null,
        "spotUsd": 0.000491934848082592,
        "refUsd": 0.00044218110309972465,
        "feesCollectedUsd": 355.6872865478012,
        "lastAction": "paused after a max-loss close until 2026-09-06T06:16Z",
        "at": 1788650150483
      },
      {
        "symbol": "NEST",
        "band": "NEST@TWO",
        "pool": "0x7686aca0b66bea34614a1a3b0b3ec158fe150b09934baa6398c98511d990e935",
        "protocol": "v4",
        "capitalUsd": 12000,
        "mode": "two-sided",
        "fee": 30000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": {
          "tokenId": "1951186",
          "lowerUsd": 0.0008698019641339316,
          "upperUsd": 0.0020146959326937867,
          "depositUsd": 5097.913021,
          "depositQty": 4116545.8003404,
          "depositStockCostUsd": 6279.639566062678,
          "depositQuoteQty": 0,
          "stockQty": 5629511.443543357,
          "cashUsd": 3252.159107890577,
          "quoteQty": 3252.159107890577,
          "valueUsd": 9594.495128760338,
          "feesUnclaimedUsd": 28.39970957493673,
          "feesCollectedUsd": 689.092957042795,
          "inRange": true,
          "openedAt": 1788642433198,
          "txHash": "0x6c19add3b3eda7c77bd757c296c90dc959d08813ab498d72679aa21bf6dc0835"
        },
        "spotUsd": 0.001138563882734398,
        "refUsd": 0.0011266228134492848,
        "feesCollectedUsd": 12574.155634125962,
        "lastAction": null,
        "at": 1788650151218
      },
      {
        "symbol": "UNIPCS",
        "band": "UNIPCS",
        "pool": "0x7b261f6244bb469568783f6ba4b6042b9d62901490d5b8501aee0c7f4e7af052",
        "protocol": "v4",
        "capitalUsd": 2000,
        "mode": "two-sided",
        "fee": 50000,
        "stockIs0": false,
        "quoteSymbol": "USDG",
        "quoteToken": "0x5fc5360d0400a0fd4f2af552add042d716f1d168",
        "maxLossBps": 6000,
        "hooked": false,
        "position": null,
        "spotUsd": 2.9389568075855845e-27,
        "refUsd": 0.001968487147011522,
        "feesCollectedUsd": 162.68922509206075,
        "lastAction": "waiting: pool price 0.0000 is 100% off the real price 0.0020; not placing until the pool is re-aligned",
        "at": 1788650151571
      }
    ],
    "feesCollectedUsd": 63418.749738553415,
    "at": 1788650151572
  }
}
```

## Appendix C — Compact summary (embedded)

```json
{
  "generatedAt": "2026-09-05T23:15:53.723247+00:00",
  "reportVersion": "v3-merged",
  "pin": {
    "block_number": 55498712,
    "block_hash": "0x905feebb6faf85315a8f2697e121f538dcc28d4e58fd8e102d59dc3c4df65823",
    "utc": "2026-09-05T23:03:40Z",
    "chain_id": 4663
  },
  "rounds": 164,
  "onchainTxs": 561,
  "multiTxRounds": 115,
  "sumListUsd": 48633.682658,
  "status": {
    "mode": "live",
    "fund": "0x0feb08fcf34f5c0e270261bc79bafc0f8eb87f98",
    "feeToken": "0xe2dae1c072b9f66bed999873b94798b2152af7e1",
    "distributedUsd": 48808.79344900001,
    "realizedProfitUsd": 49116.8582435298,
    "pendingProfitUsd": 308.06479452978965,
    "usdgBalance": 18075.664799,
    "ethBalance": "1.2677506289738811",
    "lpPrincipalPnlUsd": -36962.94028217039
  },
  "feeFlow": {
    "escrowUsd": 0,
    "claimedUsd": 159395.3490740001,
    "lastClaimAt": 1788648609645,
    "phase": 2,
    "recipientIsVault": true,
    "at": 1788650106324
  },
  "token": {
    "priceUsd": 0.0017902589788860127,
    "mcapUsd": 1790258.9788860127,
    "supply": 1000000000,
    "at": 1788650143393
  }
}
```

## Appendix D — Full distribution history (embedded)

All **164** payout rounds from `GET https://www.arbitrageape.app/api/distributions` at this report generation. Formerly a separate `distributions.json`. Each entry includes timestamp (`at`, ms), `totalUsd`, `holders`, cumulative realized, and `txHash` (comma-separated when a round spans multiple txs).

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
  }
]
```

## Appendix E — Methodology notes

- Chain reads via `https://rpc.mainnet.chain.robinhood.com`
- Function selectors resolved through OpenChain / 4byte signature DBs and runtime bytecode scan of `vault.runtime.hex`
- State pinned to block `55498712` / `0x905feebb6faf85315a8f2697e121f538dcc28d4e58fd8e102d59dc3c4df65823` / `2026-09-05T23:03:40Z`
- USDG amounts use **6 decimals**; AA uses **18**
- API `distributedUsd` can differ slightly from the sum of listed rounds — treat both as approximate desk accounting and prefer onchain sums for forensic work
- This Markdown file is **self-contained**: status, meta, full distribution history, pin, and surface ledger are embedded rather than shipped as companion JSON files
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
      "block_number": 55498712,
      "block_hash": "0x905feebb6faf85315a8f2697e121f538dcc28d4e58fd8e102d59dc3c4df65823",
      "utc": "2026-09-05T23:03:40Z"
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
  "report_source_id": "arbitrage-ape-dd-v3-merged",
  "no_real_signing": true,
  "no_broadcast": true
}
```
