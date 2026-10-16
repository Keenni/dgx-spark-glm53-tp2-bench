# DGX Spark ×2 TP2 — GLM-5.3-Flash EXL3 Benchmark

English | [简体中文](README.zh-CN.md)

## Status: withdrawn, re-measurement in progress

The comparison published here on 2026-10-16 was measured on a misconfigured stack: `TF_GLM_HC_SPLIT=1` with `TF_GLM_PREFILL_OVERLAP=0` on a TensorFold build whose `hcsplit.py` lacked the world-2 own-row fix — every prompt at or above 64 rows took the broken non-overlap row-split path and produced deterministically wrong output. The measured throughput was inflated by exactly the work that path skipped (half the partial fills), so both the prefill lead and the concurrency-decode advantage were artifacts.

The numbers have been removed; a re-measured comparison on the corrected configuration will be published here.

### Setup (unchanged)

- 2× NVIDIA DGX Spark (GB10), tensor parallel = 2
- Model: GLM-5.3-Flash (EXL3 4bpw, abliterated)
- GPU clocks: locked at 2000 MHz
- Harness: sparkDash built-in LLM benchmark
