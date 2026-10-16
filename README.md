# DGX Spark ×2 TP2 — GLM-5.3-Flash EXL3 实测对比

同一天背靠背（间隔 4 分钟）跑的 prefill / decode 对比：**自建 TensorFold 栈** vs **Mia 栈**，同一套 2×DGX Spark (GB10) TP2、同一个权重、同一个测速框架，GPU 频率锁 2000 MHz。

- 测试时间：2026-10-16 晚（两组各一次，背靠背）
- 硬件：2× NVIDIA DGX Spark (GB10)，Tensor Parallel = 2
- 模型：GLM-5.3-Flash（EXL3 量化）
- GPU 时钟：锁定 2000 MHz
- 测速：sparkDash 内置 LLM bench（同一 harness，两组参数完全一致）

## Prefill（tok/s，TTFT ms）

| input tokens | 自建栈 tps | 自建栈 TTFT | Mia 栈 tps | Mia 栈 TTFT | tps 差距 |
|---:|---:|---:|---:|---:|---:|
| 4,120  | 1549.7 | 2659 | 1516.9 | 2716 | +2.2% |
| 8,214  | 1702.0 | 4826 | 1562.3 | 5260 | +8.9% |
| 16,408 | 1774.8 | 9245 | 1610.5 | 10189 | +10.2% |
| 32,790 | 1791.5 | 18303 | 1631.5 | 20101 | +9.8% |
| 65,560 | 1762.4 | 37199 | 1619.9 | 40471 | +8.8% |

（prompt 为连续 prose，实际 prompt tokens 见行首；completion 8 tokens，只测 prefill。）

**结论：prefill 全档位自建栈领先，8k 以上稳定 +9~10%。**

## Decode（单流 tok/s，prose prompt，completion 400 tokens）

| concurrency | 自建栈 | Mia 标 | 备注 |
|---:|---:|---:|---|
| 1 | 54.1 | 54.5 | 持平 |
| 2 | 80.3 | 39.7 | 自建 ×2 |
| 3 | 82.4 | 30.1 | 自建 ×2.7 |
| 4 | 61.1 | 24.8 | 双方都回落，自建仍 ×2.5 |

单流 decode 两边打平；并发 ≥2 后 Mia 栈单流掉速明显（聚合吞吐未见优势），自建栈单流 decode 在并发下基本保持。

## 方法说明 / 注意事项

- 两组测试间隔 4 分钟、同一 bench 工具、相同 prompt 构成与长度，除引擎栈外变量已对齐。
- 每个档位只跑了 1 次，未做多次重复取方差；prefill 4k 档差距（+2.2%）在单次波动范围内，8k 以上差距更可信。
- 2000 MHz 为人为锁频。社区 reported ~2000 tok/s prefill 的数据若为更高时钟（如 2200 MHz）所得，不能直接与本文数字横比。
- decode 并发档为短输出（400 tokens）场景，不代表长生成稳态。

## 复现

- Prefill bench：input 4k/8k/16k/32k/64k tokens，prose 文本，completion = 8，记录 TTFT 与 prefill tok/s。
- Decode bench：concurrency 1–4，prose prompt，completion = 400。
