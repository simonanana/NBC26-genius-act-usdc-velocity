# Data Dictionary and Acquisition Guide

Raw inputs are **not** committed to this repository (provider terms of service).
Place the files below in `data/manual/`. The loader (`src/data_loader.py`) is
tolerant of real-world export quirks: XLSX files with a `.csv` extension,
preamble rows before the header, thousands separators, and mixed encodings.

Sample window: **2024-01-01 → 2026-03-31** (daily unless noted).
Treatment dates: **T₁ = 2025-07-18** (GENIUS Act signed), **T₂ = 2026-02-25** (OCC NPRM).

## Required Files

### 1. `artemis_usdc_velocity.csv` — H1 (treated series)

Source: Artemis Analytics. Daily EOA-to-EOA **adjusted** transfer volume
(bot/MEV/bridge-internal flows excluded; raw volume is ~1.8x the adjusted
measure) and circulating supply aggregated across canonical deployments
(Ethereum, Base, Arbitrum, Optimism, Polygon, Solana), net of burns.

| Column | Type | Description |
|---|---|---|
| `date` | date | Daily observation date |
| `usdc_adj_vol` (or `adj_vol`) | float | Adjusted transfer volume, USD |
| `usdc_supply` (or `supply`) | float | Circulating supply, USD |

Velocity is computed as `adj_vol / supply` (daily turnover multiple; a value
of 1.484 means 148.4% of outstanding supply changed hands that day).

### 2. `artemis_usdt_velocity.csv` — H1 (DiD control)

Parallel construction for USDT. Columns: `date`, `usdt_adj_vol`/`adj_vol`,
`usdt_supply`/`supply`.

## Optional Files

### 3. `protocol_tvl_timeseries.csv` — H2

Source: DefiLlama. Daily TVL for the seven protocols in the ternary panel:
Aave V3 (Lending); Uniswap V3/V4 (DEX); BUIDL, Ondo, BENJI, USYC, Mountain
USDM (RWA). Column names are auto-classified by keyword (`aave`/`lending` →
Lending; `uniswap`/`dex` → DEX; `buidl`/`ondo`/`benji`/`usyc`/`mountain`/`rwa`
→ RWA), plus a `date` column. **If this file is absent, the pipeline
automatically falls back to the public DefiLlama API** (`api.llama.fi`),
caching responses in `data/api_cache/`.

### 4. `dune_uniswap_lp_snapshot.csv` — H3

Source: Dune Analytics. LP position snapshots for the USDC/WETH 0.05% fee
tier on Uniswap V3, four cross-sections spanning June 2024 – October 2025.

| Column (auto-detected) | Description |
|---|---|
| `snapshot_date` / `date` | Snapshot date |
| `provider` / `owner` / `wallet` / `address` | LP wallet address |
| `liquidity` / `liquidity_usd` | Position liquidity (positions are summed per wallet) |
| `pool` / `fee` / `tier` (optional) | Used to filter to the 0.05% tier if present |

### 5. `macro_controls_merged.csv` — ITS controls

Source: DefiLlama + Etherscan. Columns: `date`, plus any of `log_defi_tvl`,
`log_gas`, `defi_tvl`, `eth_gas_usd`.

### 6. `aave_usdc_apy.csv` — rate-compression discussion

Source: DefiLlama `/yields`. Columns: `date`, `apy` (Aave V3 USDC supply APY).

## External Validity Check

2025 annual USDC adjusted volume in this construction totals USD 17.86T,
within 2.4% of the independently reported USD 18.3T (Bloomberg, Jan 2026);
supply reconciles within 0.4% of Circle's monthly attestations.
