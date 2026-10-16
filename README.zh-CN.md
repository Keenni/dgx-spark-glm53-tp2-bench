# DGX Spark ×2 TP2 — GLM-5.3-Flash EXL3 实测对比

[English](README.md) | 简体中文

## 状态：已撤下，重测中

2026-10-16 发布的这组对比测在了错误配置的栈上：`TF_GLM_HC_SPLIT=1` + `TF_GLM_PREFILL_OVERLAP=0`，而所用 TensorFold 构建里的 `hcsplit.py` 缺少 world-2 own-行修复——所有 ≥64 行的 prompt 都走了坏的非重叠行切分路径，确定性输出错误。其吞吐数字恰好被该路径跳过的那半工作（一半的 partial 填充）虚高，prefill 领先与并发 decode 优势都是假象。

数据已删除；修正配置后的重测对比将发布在这里。

### 测试环境（不变）

- 2× NVIDIA DGX Spark (GB10)，Tensor Parallel = 2
- 模型：GLM-5.3-Flash（EXL3 4bpw，去审查版）
- GPU 时钟：锁定 2000 MHz
- 测速：sparkDash 内置 LLM bench
