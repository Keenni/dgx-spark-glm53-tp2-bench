# DGX Spark ×2 TP2 — GLM-5.3-Flash EXL3 Benchmark

English | [简体中文](README.zh-CN.md)

Back-to-back comparison (4 minutes apart, same day) of prefill / decode throughput: **self-built TensorFold stack** vs **Mia stack**, on the same 2× DGX Spark (GB10) TP2 setup, same weights, same benchmark harness, GPU clocks capped at 2000 MHz.

- Date: evening of 2026-10-16 (one run each, back-to-back)
- Hardware: 2× NVIDIA DGX Spark (GB10), tensor parallel = 2
- Model: GLM-5.3-Flash (EXL3 quantized)
- GPU clocks: locked at 2000 MHz
- Harness: sparkDash built-in LLM benchmark (identical config for both runs)

## Prefill (tok/s, TTFT ms)

| input tokens | self-built tps | TTFT | Mia tps | TTFT | tps delta |
|---:|---:|---:|---:|---:|---:|
| 4,120  | 1549.7 | 2659 | 1516.9 | 2716 | +2.2% |
| 8,214  | 1702.0 | 4826 | 1562.3 | 5260 | +8.9% |
| 16,408 | 1774.8 | 9245 | 1610.5 | 10189 | +10.2% |
| 32,790 | 1791.5 | 18303 | 1631.5 | 20101 | +9.8% |
| 65,560 | 1762.4 | 37199 | 1619.9 | 40471 | +8.8% |

(Continuous prose prompts; actual prompt tokens in the first column; completion = 8 tokens, prefill only.)

**Result: the self-built stack leads at every context length, +9–10% from 8k up.**

## Decode (per-stream tok/s, prose prompt, completion 400 tokens)

| concurrency | self-built | Mia | notes |
|---:|---:|---:|---|
| 1 | 54.1 | 54.5 | tie |
| 2 | 80.3 | 39.7 | 2× |
| 3 | 82.4 | 30.1 | 2.7× |
| 4 | 61.1 | 24.8 | both drop, still 2.5× |

Single-stream decode is a tie; from concurrency 2 up, the Mia stack's per-stream throughput falls off sharply with no aggregate advantage, while the self-built stack holds.

## Methodology / caveats

- The two runs were 4 minutes apart, same benchmark tool, same prompt construction and lengths; the engine stack is the only variable.
- 1 run per configuration, no repeated trials / variance; the 4k gap (+2.2%) is within single-run noise, the 8k+ gaps are more trustworthy.
- 2000 MHz is a manual cap. Community-reported ~2000 tok/s prefill figures, if taken at higher clocks (e.g. 2200 MHz), are not directly comparable to these numbers.
- Decode concurrency is a short-output (400 tokens) scenario; it does not represent long-generation steady state.

## Reproducing

- Prefill bench: input 4k/8k/16k/32k/64k tokens, prose text, completion = 8; record TTFT and prefill tok/s.
- Decode bench: concurrency 1–4, prose prompt, completion = 400.
