# Dual-Track Weekly Plan

> **H100：把 DyLLM 官方 baseline 复现干净。**
> **5090：验证 partial / key-delta 核心假设。**

两边不要混任务。H100 不做探索，5090 不追最终系统测速。

| Machine | Role | Branch | Goal this week |
|---------|------|--------|----------------|
| **H100** | reproduction / system truth | `repro-h100` | Official DyLLM baseline reproduction |
| **5090** | research / fast iteration | `partial-5090` | Validate middle-band partial-update hypothesis |

**Shared baseline tag:** [`baseline-dyllm-repro-v1`](https://github.com/MSA-yd/DyLLM/tree/baseline-dyllm-repro-v1) → commit `20850b0`

**Weekend deliverables:**

- [`docs/templates/h100_reproduction.md`](templates/h100_reproduction.md)
- [`docs/templates/partial_oracle.md`](templates/partial_oracle.md)

---

# 一、H100 本周任务：复现线

## 本周目标

到周末，应能回答三个问题：

1. DyLLM 的 **accuracy 能不能在 H100 上复现**？
2. DyLLM 的 **speedup 能不能接近论文**？
3. 5090 上 τ=0.99 掉点，是环境问题还是这个 operating point 本身不稳定？

论文是在单张 H100 PCIe 80GB 上测试，并使用 custom CUDA sparse-attention/cache kernel，所以 H100 才是最终判断系统结果的主要依据。

---

## H100-P0：环境完全记录

第一件事不要跑实验，先保存：

```text
GPU model
PCIe / SXM
driver
CUDA
PyTorch
flash-attn
gcc
nvcc
DyLLM git commit / tag
model checkpoint
```

输出：

```text
environment_h100.txt
```

同时确认：

* custom CUDA extension 编译成功；
* 实际走的是作者 sparse kernel；
* 没有 silent fallback 到 PyTorch implementation。

---

## H100-P0：只先跑 GSM8K 三组

保持和 5090 完全相同的 evaluation pipeline。

> 5090 reference（dirty tree，仅作对照，非正式终稿）：
> `/mnt/sda/qluo/myd/DyLLM_experiments/paper_table2_5090/gsm8k/`

### Experiment H1 — Original

| Source | Acc (flexible) |
|--------|---------------:|
| Paper | 77.79% |
| 5090 | 78.70% |

记录：

* flexible-extract
* strict
* tokens/s
* peak memory
* wall-clock time

### Experiment H2 — DyLLM (τ=0.995)

| Source | Acc | Speedup |
|--------|----:|--------:|
| Paper | 78.01% | 6.99× |
| 5090 | 78.39% | 2.61× |

这是最重要的 anchor。

### Experiment H3 — DyLLM (τ=0.99)

| Source | Acc | Speedup |
|--------|----:|--------:|
| Paper | 79.08% | 7.60× |
| 5090 | 77.10% | 2.88× |

这组是本周最需要解释的 discrepancy。

**Recommended H100 eval flags (official path):**

```bash
# Original: threshold>1 so every token is salient (no FFN reuse)
--experiment-mode disabled --threshold 2.0

# DyLLM
--experiment-mode disabled --threshold 0.995   # or 0.99
--num-shot 5 --num-steps 256 --num-full-steps 4 --block-size 32
--batch-size 1 --temperature 0 --seed 1234
```

Do **not** use research modes (`oracle_three_way`, pipeline `binary`/`full`) for the official Table 2 numbers.

---

## H100-P0：吞吐重复测

accuracy 跑完整 dataset 一次即可。

throughput 至少 **3** 次，最好 **5** 次。

最后报告：

```text
median tokens/s
min
max
speedup vs H100 Original
```

不要拿 H100 absolute tokens/s 和 5090 absolute tokens/s 直接比。

只比较各自 GPU 上的：

\[
\frac{\text{DyLLM}}{\text{Original}}
\]

---

## H100-P1：跨 GPU saliency diagnostic

不用 dump hidden state。

对同样一小批（GSM8K 20–50 题），记录每个 \((t,l)\) 下的 salient token indices，比较：

\[
A^{H100}_{t,l}
\quad\text{vs}\quad
A^{5090}_{t,l}.
\]

Jaccard：

\[
J=
\frac{|A_H\cap A_{5090}|}
{|A_H\cup A_{5090}|}.
\]

解读：

| Observation | Interpretation |
|-------------|----------------|
| Acc differs, salient sets almost same | FP / kernel numerical difference amplified by denoising trajectory |
| Salient sets diverge early | τ=0.99 threshold decision is numerically sensitive |

这个结果以后可能成为 motivation 的辅助证据。

---

## H100-P1：GSM8K 对齐后再补其他任务

若 H1–H3 基本正常，再跑：

1. MATH
2. MBPP
3. MMLU-Pro

本周不要求全跑完。优先保证 **GSM8K reproduction 干净**。

---

## H100 本周停止条件

### Case A：accuracy 对，speed 不对

开始 profile：

* sparse kernel 是否生效；
* batch size；
* synchronization；
* warmup；
* FlashAttention；
* PCIe/SXM；
* 是否统计了额外 CPU/eval 时间。

### Case B：Original 和 .995 对，.99 仍明显掉点

不要反复调到 79.08。记录成 observation。

这很可能意味着 `.99` operating point 不稳定——对 partial-update story 是好事。

---

# 二、5090 本周任务：研究线

## 本周目标

只回答一个核心问题：

> **DyLLM 中 full/reuse 二元决策之间，是否真的存在一个有价值的 partial-update 区域？**

最值得研究的区间：

\[
\boxed{0.99 \le s < 0.995}
\]

原因：

* 在 τ=.995 下，这批 token → **Full**
* 在 τ=.99 下，这批 token → **Reuse**
* accuracy：78.39 → 77.10

它们不能简单全部 reuse。自然问题：

> 是否不需要 full，只需要 partial？

---

## 5090-P0：先统计 middle band 规模

在 GSM8K 50–100 道样本上统计：

\[
\rho_{t,l}
=
\frac{
|\{i:0.99\le s_{t,l,i}<0.995\}|
}{N}.
\]

至少生成：

* **Figure A** — middle-band token ratio vs layer
* **Figure B** — middle-band token ratio vs denoising step
* **Figure C** — layer × step heatmap

若只有 ~1%，系统价值有限；若 10–30%，非常值得做。

---

## 5090-P0：Alpha Oracle（本周最重要实验）

先不要碰 SVD。

定义：

\[
\Delta h = h^{\text{full}} - h^{\text{cache}}.
\]

对 middle-band token：

\[
h^{\text{partial}} = h^{\text{cache}} + \alpha\Delta h.
\]

扫 \(\alpha \in \{0,\ 0.25,\ 0.5,\ 0.75,\ 1\}\)。

其他 token：

* \(s \ge 0.995\) → reuse
* \(s < 0.99\) → full

目标表：

| Middle band | Acc |
|-------------|----:|
| reuse (α=0) | 77.10 |
| α=.25 | ? |
| α=.50 | ? |
| α=.75 | ? |
| full (α=1) | 78.39 |

---

## 5090-P0：不要只看最终 accuracy

每个 α 同时记录：

### Hidden fidelity

\[
\cos(h^{\text{partial}}, h^{\text{full}})
\]

### Logit fidelity

\[
\mathrm{KL}(p_{\text{full}} \Vert p_{\text{partial}})
\]

### Decoding fidelity

* top-1 token agreement
* unmask-position agreement
* first divergence step

### Final task

GSM8K flexible-extract

即使最终 accuracy 只差 0.3pp，也能看到 trajectory 如何变化。

---

## 5090-P0：paired answer flip

对 `τ=.99 reuse` vs `partial` vs `τ=.995 full` 统计：

| Base | Partial | Count |
|------|---------|------:|
| wrong | correct | |
| correct | wrong | |
| correct | correct | |
| wrong | wrong | |

尤其人工看 **reuse 错 → partial 对** 的 case，判断 partial 是在恢复必要 delta，还是随机改变轨迹。

---

## 5090-P1：Alpha 有效之后再做 SVD

仅当发现 α=.25/.5 已能恢复大部分 full performance 时，再问：

> **应该保留 delta 的哪些方向？**

对 middle-band delta \(\Delta H = U\Sigma V^\top\)，跑：

\[
r/d \in \{1/64, 1/32, 1/16, 1/8, 1/4, 1/2\}.
\]

至少比较三种：

| Variant | Construction |
|---------|--------------|
| Top-r | \(U_r\Sigma_r V_r^\top\) |
| Random-r | same rank, random subspace |
| Bottom-r | lowest singular directions |

### SVD 判断标准

真正需要看到的不是 “top-r reconstruction error 很小”，而是在

* logit KL
* decoding agreement
* final accuracy

上成立：

\[
\text{Top-r} > \text{Random-r} > \text{Bottom-r}
\]

否则只能说 delta 可压缩，不能说存在 **key delta**。

---

## 5090-P1：layer-wise / block-wise 分析

若 SVD 有效，再测：

* **Effective rank by layer** — 例如 \(r_{90}\)（90% delta energy）
* **Subspace similarity across layers** — \(V_l \leftrightarrow V_{l+1}\)

若连续 2–4 层很相似 → block-wise shared basis 值得做；差异明显 → 坚持 layer-wise。

对应导师问题：每层单独看还是几层综合起来看。

---

# 三、本周日程

## H100

| Day | Task |
|-----|------|
| 1 | 环境 + CUDA kernel sanity check |
| 2 | Original GSM8K + throughput repeats |
| 3 | τ=.995 |
| 4 | τ=.99 |
| 5 | 5090/H100 salient-set comparison + discrepancy analysis |
| 6–7 | 对齐则补 MATH/MBPP；未对齐则 profile/debug，不盲目跑新 benchmark |

## 5090

| Day | Task |
|-----|------|
| 1 | 统计 `.99–.995` middle-band 分布 |
| 2 | 实现 alpha partial oracle |
| 3 | 跑 α=0,.25,.5,.75,1 |
| 4 | hidden/logit/decoding analysis + paired flips |
| 5 | alpha 结果 positive → Top-r SVD |
| 6 | Random-r / Bottom-r |
| 7 | layer/step effective rank + 汇总 |

---

# 四、本周明确不要做

## H100 不做

* trajectory 大规模 dump
* SVD
* predictor training
* partial update
* 大量 ablation
* 修改官方算法

## 5090 不做

* 最终 CUDA 优化
* 为了 tokens/s 重写 kernel
* 同时做 Dream/MATH/MBPP
* 训练 complicated predictor
* 同时尝试 5 种 partial 方法

这周先把 **oracle story** 弄明白。

---

# 五、周末两份产物

填好后放到 `docs/results/`（或各机实验目录），并尽量把 summary 同步回本仓库。

1. [`h100_reproduction.md`](templates/h100_reproduction.md) — reproduction passed / partially passed / discrepancy remains
2. [`partial_oracle.md`](templates/partial_oracle.md) — middle-band / α / SVD / layer-vs-block 结论

---

# 六、本周两条生死线

### H100

> **`.99` discrepancy 到底是不是可复现的？**

### 5090

> **`.99–.995` 这批 token 能不能不用 full、只做 partial？**

只要第二个问题答案是明显 **Yes**，课题就已经从“想法”进入一个非常具体、值得继续追的方法问题。

---

# Sync checklist（两边共用）

每个 run 目录至少保存：

```text
run_config.yaml / config.json
environment.json   # git_commit, git_dirty, gpu, driver, cuda, torch, flash_attn, ...
lm_eval_results.json
summary.json
run.log
```

硬规则：

* `role=repro` 且 `git_dirty=true` → 拒绝作为官方复现数字
* H100 只 checkout `repro-h100`（或 `baseline-dyllm-repro-v1`）
* 5090 research 改动只进 `partial-5090`，不要推到 `repro-h100`
