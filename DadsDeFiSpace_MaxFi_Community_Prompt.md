# DadsDeFiSpace — MaxFi Dashboard Community Prompt
## Screenshot your MaxFi → Paste to Claude → Get JSON → Load your dashboard

**Dashboard:** https://dadsdefi1013.github.io/maxfiportfoliotracker/
**Community:** t.me/DADSDefiSpace · @cryptozone1013 · dadsdefispace.org

---

## HOW TO USE

**Option A — Screenshots (easiest):**
1. Open MaxFi at maxfi.tech and go to your Positions page
2. Screenshot each position card (expand fully so all data is visible)
3. Open Claude at claude.ai — paste this entire prompt, then attach your screenshots
4. Claude reads the images and returns your JSON

**Option B — Copy/paste text:**
1. Select all text on your MaxFi positions page and copy it
2. Open Claude at claude.ai — paste this entire prompt, then paste your MaxFi text below it

**Then:**
1. Copy the JSON Claude returns
2. Go to: https://dadsdefi1013.github.io/maxfiportfoliotracker/
3. Paste into the input box → click **Generate dashboard**
4. All 7 tabs render instantly

---

## ════════════════════════════════════════════════
## PASTE THIS ENTIRE BLOCK INTO CLAUDE
## ════════════════════════════════════════════════

You are extracting MaxFi DeFi position data and formatting it as JSON for the DadsDeFiSpace MaxFi Portfolio Dashboard.

**Return ONLY a valid JSON array. No explanation, no markdown code fences, no preamble. Start with [ and end with ].**

---

### REQUIRED FIELDS — extract these for every position

**Identity**
- `pair` — token pair exactly as shown (e.g. "WETH/cbBTC", "SOL/WETH", "USDC/cbBTC", "AERO/cbBTC")
- `positionId` — the # number on the card as a STRING (e.g. "2048441")
- `dex` — "PancakeSwap" if ID is 7 digits starting with 2; "Aerodrome" if ID starts with 7
- `daysOld` — integer days shown (e.g. "21d" = 21, "3d 15h" = 3)

**Value & Earnings**
- `value` — Position Value in USD (number, no $ sign)
- `earned` — Total Earned in USD (number)
- `uncollected` — uncollected/pending fees in USD (number; 0 if not shown)
- `collected` — collected fees in USD (= earned − uncollected)
- `apr` — Earnings Rate % shown (e.g. 107.8)
- `dpd` — $/day projected (number next to Earnings Rate, e.g. 0.51)
- `rebalances` — Rebalances count (integer)

**Status flags**
- `inRange` — true if "In Range", false if "Out of Range"
- `autoCompound` — true if "Compound: Enabled", false if "Compound: Disabled"

**initialValue — CRITICAL for performance metrics**
- `initialValue` — USD value of this position when it was FIRST OPENED
  - New position opened today → use current `value`
  - Known from previous snapshot → use that number
  - Unknown → set to 0 (all other metrics still work)
  - This powers the vsETH and vsHODL accumulation calculations (Agent Max methodology)

**Token data**
- `tokenA` — first token symbol (e.g. "WETH", "USDC", "SOL", "AERO", "cbBTC")
- `qtyA` — current quantity of tokenA (number)
- `entryPriceA` — USD price of tokenA when position opened (0 if unknown)
- `currentPriceA` — current USD price of tokenA (derive using rules below)
- `tokenB` — second token symbol
- `qtyB` — current quantity of tokenB (number; use 0.0 when position is out of range and holding only one token)
- `entryPriceB` — USD price of tokenB when position opened (0 if unknown)
- `currentPriceB` — current USD price of tokenB (derive using rules below)

**Range**
- `rangeLower` — lower bound of price range (exact number as shown)
- `rangeUpper` — upper bound of price range (exact number as shown)
- `rangeCurrentPrice` — current price shown in the range (exact number as shown)

**Optional**
- `walletAddress` — wallet address string if you want it shown in the header

---

### PRICE DERIVATION RULES

**1. Stablecoins = $1.00 always**
USDC, USDT, DAI, FRAX, EURC → currentPrice = 1.00

**2. USDC/cbBTC**
The range price shown IS the cbBTC USD price directly.
e.g. current price $59,740 → cbBTC = $59,740 · USDC = $1.00

**3. WETH/cbBTC** — ratio pair (cbBTC per WETH)
WETH_USD = cbBTC_USD × ratio
e.g. ratio 0.026273 and cbBTC = $59,401 → WETH = 59401 × 0.026273 = $1,560.65

**4. SOL/WETH** — ratio pair (WETH per SOL)
SOL_USD = WETH_USD × ratio
e.g. ratio 0.043882 and WETH = $1,553 → SOL = $68.15

**5. WETH/uSOL** — ratio pair (uSOL per WETH)
uSOL_USD = WETH_USD ÷ ratio
e.g. WETH = $1,553 and ratio = 22.85 → uSOL = $67.97

**6. WETH/VVV** — ratio pair (VVV per WETH)
VVV_USD = WETH_USD ÷ ratio
e.g. WETH = $1,553 and ratio = 115.2 → VVV = $13.48

**7. AERO/cbBTC** — ratio pair (cbBTC per AERO)
Solve from position value: aero_price = (value − qtyB × cbBTC_price) ÷ qtyA
e.g. value = $69, 72.1 AERO + 0.000585 cbBTC at $59,740 → (69 − 34.95) ÷ 72.1 = $0.4723

**8. Out of range positions** — hold only ONE token
The position shows only one token quantity. Use position value ÷ quantity to derive price.
e.g. "0.154 WETH" OOR and value = $239.55 → WETH = $239.55 ÷ 0.154 = $1,555
Set the other token's qty to 0.0

**9. Unknown price** → set currentPriceA and currentPriceB to 0
The dashboard still works — only IL/vsETH calculations show "n/a"

---

### OUT OF RANGE DETECTION

When a position shows "Out of Range":
- `inRange` = false
- The card shows only ONE token (fully converted)
- Set the absent token's `qty` to 0.0
- `rangeCurrentPrice` will be OUTSIDE the rangeLower–rangeUpper band

Examples:
- SOL/WETH out of range ABOVE upper → shows "0.154 WETH" → tokenA:"WETH", qtyA:0.154, tokenB:"SOL", qtyB:0.0
- USDC/cbBTC out of range BELOW lower → shows "0.00272 cbBTC" → tokenA:"cbBTC", qtyA:0.00272, tokenB:"USDC", qtyB:0.0

---

### DEX DETECTION

- Position ID starts with 2 (7 digits, e.g. 2048441, 2050636) → `"PancakeSwap"`
- Position ID starts with 7 (8 digits, e.g. 72276604, 72487068) → `"Aerodrome"`

---

### DEFAULTS FOR MISSING DATA

| Field | Default |
|-------|---------|
| positionId | "unknown" |
| daysOld | 0 |
| rebalances | 0 |
| uncollected | 0 |
| collected | same as earned |
| initialValue | 0 |
| entryPriceA/B | 0 |
| currentPriceA/B | 0 |
| autoCompound | true |

---

### COMPLETE EXAMPLE (Jun 28, 2026 — 6 positions)

```json
[
  {
    "pair": "SOL/WETH",
    "positionId": "2050636",
    "dex": "PancakeSwap",
    "daysOld": 2,
    "initialValue": 262.16,
    "value": 239.55,
    "earned": 1.65,
    "uncollected": 0.00,
    "collected": 1.65,
    "apr": 125.0,
    "dpd": 0.82,
    "rebalances": 1,
    "inRange": false,
    "autoCompound": true,
    "tokenA": "WETH",
    "qtyA": 0.154,
    "entryPriceA": 1553.05,
    "currentPriceA": 1553.05,
    "tokenB": "SOL",
    "qtyB": 0.0,
    "entryPriceB": 68.15,
    "currentPriceB": 68.15,
    "rangeLower": 0.042505,
    "rangeUpper": 0.043843,
    "rangeCurrentPrice": 0.043882,
    "walletAddress": "0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0"
  },
  {
    "pair": "USDC/cbBTC",
    "positionId": "2050433",
    "dex": "PancakeSwap",
    "daysOld": 17,
    "initialValue": 175.00,
    "value": 162.62,
    "earned": 7.73,
    "uncollected": 0.12,
    "collected": 7.61,
    "apr": 99.7,
    "dpd": 0.44,
    "rebalances": 8,
    "inRange": true,
    "autoCompound": true,
    "tokenA": "USDC",
    "qtyA": 33.1,
    "entryPriceA": 1.00,
    "currentPriceA": 1.00,
    "tokenB": "cbBTC",
    "qtyB": 0.00217,
    "entryPriceB": 59740.0,
    "currentPriceB": 59740.0,
    "rangeLower": 59258.0,
    "rangeUpper": 61676.0,
    "rangeCurrentPrice": 59740.0,
    "walletAddress": "0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0"
  },
  {
    "pair": "WETH/cbBTC",
    "positionId": "2042097",
    "dex": "PancakeSwap",
    "daysOld": 22,
    "initialValue": 160.67,
    "value": 147.46,
    "earned": 5.50,
    "uncollected": 2.83,
    "collected": 2.67,
    "apr": 62.8,
    "dpd": 0.25,
    "rebalances": 12,
    "inRange": false,
    "autoCompound": true,
    "tokenA": "WETH",
    "qtyA": 0.0951,
    "entryPriceA": 1776.65,
    "currentPriceA": 1553.05,
    "tokenB": "cbBTC",
    "qtyB": 0.0,
    "entryPriceB": 66000.0,
    "currentPriceB": 59740.0,
    "rangeLower": 0.026038,
    "rangeUpper": 0.027372,
    "rangeCurrentPrice": 0.025952,
    "walletAddress": "0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0"
  },
  {
    "pair": "WETH/uSOL",
    "positionId": "72276604",
    "dex": "Aerodrome",
    "daysOld": 6,
    "initialValue": 119.02,
    "value": 106.37,
    "earned": 1.46,
    "uncollected": 1.16,
    "collected": 0.30,
    "apr": 82.9,
    "dpd": 0.24,
    "rebalances": 1,
    "inRange": false,
    "autoCompound": false,
    "tokenA": "WETH",
    "qtyA": 0.0686,
    "entryPriceA": 2450.0,
    "currentPriceA": 1553.05,
    "tokenB": "uSOL",
    "qtyB": 0.0,
    "entryPriceB": 102.64,
    "currentPriceB": 67.97,
    "rangeLower": 23.57,
    "rangeUpper": 24.53,
    "rangeCurrentPrice": 22.85,
    "walletAddress": "0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0"
  },
  {
    "pair": "AERO/cbBTC",
    "positionId": "72487068",
    "dex": "Aerodrome",
    "daysOld": 0,
    "initialValue": 69.00,
    "value": 69.00,
    "earned": 0.02,
    "uncollected": 0.02,
    "collected": 0.00,
    "apr": 1324.1,
    "dpd": 2.50,
    "rebalances": 0,
    "inRange": true,
    "autoCompound": false,
    "tokenA": "AERO",
    "qtyA": 72.1,
    "entryPriceA": 0.4723,
    "currentPriceA": 0.4723,
    "tokenB": "cbBTC",
    "qtyB": 0.000585,
    "entryPriceB": 59740.0,
    "currentPriceB": 59740.0,
    "rangeLower": 0.000008,
    "rangeUpper": 0.000008,
    "rangeCurrentPrice": 0.000008,
    "walletAddress": "0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0"
  },
  {
    "pair": "WETH/VVV",
    "positionId": "72423501",
    "dex": "Aerodrome",
    "daysOld": 13,
    "initialValue": 56.14,
    "value": 65.73,
    "earned": 8.31,
    "uncollected": 1.55,
    "collected": 6.77,
    "apr": 353.9,
    "dpd": 0.64,
    "rebalances": 5,
    "inRange": false,
    "autoCompound": true,
    "tokenA": "WETH",
    "qtyA": 0.0424,
    "entryPriceA": 2400.0,
    "currentPriceA": 1553.05,
    "tokenB": "VVV",
    "qtyB": 0.0,
    "entryPriceB": 20.46,
    "currentPriceB": 13.48,
    "rangeLower": 115.6,
    "rangeUpper": 125.2,
    "rangeCurrentPrice": 115.2,
    "walletAddress": "0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0"
  }
]
```

---

### BEFORE RETURNING — CHECK THESE

1. Total value of all positions ≈ MaxFi "Total Value" header
2. Total earned ≈ MaxFi "Total Earnings" header
3. Every position has: pair, value, earned, apr, dpd, tokenA, tokenB, qtyA, qtyB, rangeLower, rangeUpper, rangeCurrentPrice
4. `inRange: false` for any position showing "Out of Range" or a countdown timer
5. `autoCompound` matches what the card shows (Enabled vs Disabled)
6. JSON is valid — no trailing commas, strings quoted, numbers unquoted

---

[PASTE YOUR MAXFI SCREENSHOTS OR COPIED TEXT HERE]

## ════════════════════════════════════════════════
## END OF PROMPT
## ════════════════════════════════════════════════

---

## AFTER CLAUDE RETURNS YOUR JSON

1. Copy the entire JSON (starts with `[`, ends with `]`)
2. Go to: **https://dadsdefi1013.github.io/maxfiportfoliotracker/**
3. Paste into the input box → click **Generate dashboard**

**7 tabs render instantly:**
- **Overview** — KPIs including ETH/BTC portfolio value, vsETH, vsHODL
- **Positions** — Full Agent Max card per position (vsETH, cover ratio, paper IL, benchmarks)
- **Cashflow** — 50/50 compounding split, real APR vs shown APR
- **Risk** — Low/Mid/High tier, health scores, scorecard
- **Allocation** — Asset class breakdown (BTC, ETH, SOL, Stables)
- **Scenarios** — Bearish / Base / Bullish projections
- **Performance** — Historical USD / ETH / BTC charts, vsETH & vsBTC accumulation

---

## ABOUT initialValue

This is the most important optional field. It's the USD value of the position on the day you opened it — what MaxFi showed right after deposit.

**Why it matters:** The dashboard uses the Agent Max v11-2 methodology to calculate vsETH — how much better your LP did vs simply holding ETH. This requires your original capital (V_entry), not the current value.

**Where to find it:**
- Your first snapshot for that position in this tool's spreadsheet
- The transaction value in your wallet history on Basescan
- MaxFi's deposit confirmation screen

**If you set it to 0:** The dashboard falls back to reconstructing it from entry prices × current quantities. This is accurate for new positions but drifts after multiple rebalances.

---

## TROUBLESHOOTING

**"JSON parse error"** — Run your output through jsonlint.com. Most common: missing comma between positions, or unquoted string value.

**"Missing required fields"** — Dashboard will tell you exactly which field is missing on which position.

**"n/a" on vsETH / vsHODL** — Entry prices or initialValue not set. All other metrics still work.

**Numbers don't match MaxFi** — Check that `collected + uncollected = earned` exactly.

**Out of range IL looks wrong** — When OOR, set the absent token qty to 0.0 and use the position value to derive the remaining token's price.

---

## WEEKLY WORKFLOW (under 5 minutes)

1. Open MaxFi → screenshot all position cards
2. Open new Claude chat at claude.ai
3. Paste this prompt + attach screenshots
4. Copy the JSON Claude returns
5. Go to the dashboard → click **New snapshot** → paste → Generate
6. Share with the community

---

## LINKS

- 📊 **Dashboard:** https://dadsdefi1013.github.io/maxfiportfoliotracker/
- 🚀 **MaxFi (Kevin's referral):** https://www.maxfi.tech/deposit?ref=0x1b8021D5fb5fDcc7E1C084c042F7e765EC31a3E0
- 📢 **Telegram:** https://t.me/DADSDefiSpace
- 🐦 **X:** https://x.com/cryptozone1013
- 📚 **Free DeFi Course:** https://www.dadsdefispace.org/challenges
- ✍️ **Newsletter:** https://paragraph.com/@daddefispace
- 🟣 **Farcaster:** https://farcaster.xyz/thecaptain1013
- 🔵 **Base App:** https://base.app/profile/dadsdefispace

---

*Built by DadsDeFiSpace · Education-first DeFi · Not financial advice · Always DYOR*
*Bitcoin is the foundation. DeFi is the amplifier. Process is the edge.*
