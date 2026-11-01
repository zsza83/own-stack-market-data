# own-stack-market-data

Debate / dockit for an **own-built** market-data store aimed at daytrade / HFT-minded bots — not a finished product SKU.

**Live article (canonical narrative):**  
https://vpnstronger.com/own-stack-market-data-for-daytrade-bots-json-is-too-slow-what-instead

**Sister note (15s/30s candles):**  
https://github.com/zsza83/homemade-15s-30s-candles

## Stance

- **94 is not 100** — collect architecture material; measure; leave fake certainty on the shelf.
- Self-built corpus (WS ingest → our files), not "only mirror Binance warehouse."
- JSON is fine for configs / AI blueprints; wrong durable format on a tight hot path.
- MD4/MD5 = integrity hashes, not storage formats.

## Format shortlist (for argument)

| Format | Fit |
|--------|-----|
| **Parquet** | Cold history / backtests / hourly rollups |
| **Zarr / HDF5** | Chunked research arrays |
| **FlatBuffers / Protobuf** | Wire / IPC / realtime frames |
| **QuestDB / ClickHouse / Timescale** | Hybrid live append + time queries |
| **Raw binary + mmap** | Hot log before compaction |

**Hypothesis:** never block the trade loop on Parquet; background consumer → TSDB or binary frames; hour/day end → Parquet (or Zarr) for Polars/Python research.

## Threads & languages

- Ring buffer / lock-free queue: network enqueue → trade consumer (RAM only) → writer consumer (async flush).
- Hot path: **Rust / C++** (no GC jitter). Python for sealed-file research, not as the sole realtime engine.

## Strawman (debate only)

1. WS/FIX → Rust/C++ ingest thread
2. Lock-free queue → momentum / SEMA / trend in memory
3. Background flush → QuestDB/ClickHouse **or** framed binary log
4. Compaction → Parquet / Zarr
5. Polars/Pandas reads only sealed files

## Open questions

1. Self-built 1h OHLCV that later grows to tick: Parquet-only cold store, or Zarr/HDF5 for matrix features?
2. FlatBuffers: wire-only, or also on-disk hot log?
3. TSDB in-process vs sidecar on a small VPS — worth the RAM?
4. Ring under burst: drop oldest, or pause ingest?
5. Where does daytrade robot stop and true colocated HFT begin (NIC / kernel bypass)?
6. Does the [15s/30s homemade path](https://github.com/zsza83/homemade-15s-30s-candles) already imply this split, or are we still too JSON-shaped?

Use **Discussions** for why-not-this-way replies. No tokens, no secrets, no exploit cookbooks.

## License / tone

Public dockit for peer review. Trading involves risk. Not financial advice. Not a hosted HFT service.
