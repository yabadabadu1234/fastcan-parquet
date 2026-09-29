# fastcan-parquet


# fastcan (can2parquet)

**Ultra-fast, zero-copy, memory-efficient CAN/CAN-FD binary log to Apache Parquet converter.**

Engineered for automotive, robotics, and industrial telemetry pipelines where multi-gigabyte logs must be ingested into modern analytics (Pandas, Polars, DuckDB, PyTorch) in seconds—without running out of RAM.

---

## The Problem: Why Existing Tools Fail in Production

Most telemetry pipelines rely either on monolithic €5,000/seat desktop suites or naive Python scripts that break on real-world datasets:

1. **Out-of-Memory (OOM) Crashes:** Processing a 10 GB binary log with standard Python parsers frequently demands 30+ GB of RAM due to object boxing and unbuffered loading.
2. **Sparse/Asynchronous Matrix Bloat:** Industrial buses stream signals at differing frequencies (e.g., wheel speed at 100 Hz, battery cell temp at 1 Hz). Naive converters fill asynchronous gaps with massive `NaN` matrices, blowing up file footprints.
3. **Single-Thread Bottlenecks:** Legacy tools process frames sequentially on a single CPU core, taking 30–60 minutes per road test session.
4. **Cloud / CI/CD Incompatibility:** Proprietary vendor software is tied to Windows licensing dongles, preventing automated ingestion in Linux Docker containers or headless servers.

---

## The Solution: Core Architecture & Invariants

`fastcan` is a standalone, dependency-free command-line tool and C-ABI engine built for throughput:

- **Zero-Copy Streaming:** Utilizes memory-mapped I/O (`mmap`) to decode raw frames directly from disk buffers into Apache Arrow columnar batches.
- **Constant Memory Footprint:** Operates strictly within a user-configurable memory limit (e.g., max 512 MB RAM) regardless of whether the input file is 1 GB or 100 GB.
- **Multithreaded Block Partitioning:** Automatically splits raw binary streams into frame-aligned chunks across all available hardware threads.
- **Lossless Columnar Compression:** Direct emission of Snappy/Zstandard-compressed Apache Parquet files, reducing raw storage by up to 85% compared to CSV/MDF4 while offering instant predicate pushdown in Polars/DuckDB.

---

## Technical Specifications (Target Pipeline)

- **Input:** Raw CAN / CAN-FD binary logs (`.bin`, `.log`, `.blf` subset) + DBC specification file.
- **Output:** Snappy-compressed Apache Parquet (`.parquet`) with strictly typed columns:
  - `timestamp_us` (UInt64 / Timestamp)
  - `channel` (UInt8)
  - `message_id` (UInt32)
  - Decoded signals (Physical float64/int64 mapped via DBC scale/offset factors).
- **Core Engine:** Written in Rust / C++20.
- **Bindings:** Standalone CLI executable + zero-overhead Python wheel (`import fastcan`).

---

## Usage

```bash
# Convert a multi-gigabyte log in seconds using all CPU cores
fastcan convert \
  --input test_drive_run01.bin \
  --dbc vehicle_network.dbc \
  --output test_drive_run01.parquet \
  --threads max \
  --compression snappy

Benchmark Targets
| Dataset Size | Legacy Python Tools | Commercial Desktop GUI | fastcan (Target) |
|---|---|---|---|
| 1 GB (CAN 2.0B) | ~180 sec (OOM Risk) | ~45 sec | < 3 sec |
| 10 GB (CAN-FD) | Failed (Crash) | ~8 min | < 25 sec |
| Peak RAM Usage | > 16 GB | ~4 GB | < 512 MB |
License
Proprietary / Dual Licensing (Commercial License for Industrial Telemetry Ingestion; Community Edition for Non-Commercial Benchmarks).

