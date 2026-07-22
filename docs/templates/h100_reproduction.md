# H100 Reproduction Report

> Fill this by weekend. Machine: H100 only. Branch: `repro-h100` / tag `baseline-dyllm-repro-v1`.

## Environment

| Item | Value |
|------|-------|
| GPU model | |
| PCIe / SXM | |
| Driver | |
| CUDA | |
| PyTorch | |
| flash-attn | |
| gcc | |
| nvcc | |
| DyLLM git commit | |
| DyLLM tag | `baseline-dyllm-repro-v1` |
| Model checkpoint | |
| Sparse CUDA kernel active? | yes / no |
| Silent PyTorch fallback? | yes / no |
| Run dir | |
| Date | |

Attach / link: `environment_h100.txt`

---

## GSM8K Results (flexible-extract primary)

| Method | Paper Acc | H100 Acc | 5090 Acc | Paper Speedup | H100 Speedup | 5090 Speedup |
|--------|----------:|---------:|---------:|--------------:|-------------:|-------------:|
| Original | 77.79 | | 78.70 | 1× | | 1× |
| τ=0.995 | 78.01 | | 78.39 | 6.99× | | 2.61× |
| τ=0.99 | 79.08 | | 77.10 | 7.60× | | 2.88× |

### Throughput repeats (H100)

| Method | n | median tok/s | min | max | speedup vs H100 Original |
|--------|--:|-------------:|----:|----:|-------------------------:|
| Original | | | | | 1.00× |
| τ=0.995 | | | | | |
| τ=0.99 | | | | | |

Notes on measurement (batch size, warmup, whether eval overhead excluded):

-

---

## τ=0.99 discrepancy analysis

| Question | Answer |
|----------|--------|
| Is `.99` acc drop reproducible on H100? | yes / no / unclear |
| H100 `.99` vs paper | |
| H100 `.99` vs H100 `.995` | |
| Salient-set Jaccard (H100 vs 5090), mean | |
| Salient sets diverge early? | yes / no |

Interpretation:

-

---

## Stop-condition case

- [ ] Case A: accuracy OK, speed wrong → profile notes below
- [ ] Case B: Original & `.995` OK, `.99` still drops → recorded as observation (do not force-fit 79.08)
- [ ] Neither — reproduction looks clean

Profile / debug notes:

-

---

## Other tasks (optional this week)

| Task | Status | Acc | Notes |
|------|--------|----:|-------|
| MATH | not started / running / done | | |
| MBPP | not started / running / done | | |
| MMLU-Pro | not started / running / done | | |

---

## One-line conclusion

> reproduction passed / partially passed / discrepancy remains

Detail:

-
