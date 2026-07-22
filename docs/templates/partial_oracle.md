# Partial Oracle Report (5090 Research)

> Fill this by weekend. Machine: 5090 only. Branch: `partial-5090`.  
> Core question: does a valuable partial-update region exist between full and reuse?

**Focus band:** \(0.99 \le s < 0.995\)

---

## Middle-band scale

| Metric | Value |
|--------|------:|
| Dataset / n | GSM8K / |
| Mean middle-band ratio ρ | |
| Layer with max ρ | |
| Step with max ρ | |
| System-relevant? (≥ ~10%) | yes / no |

Figures (link paths):

- Figure A (ratio vs layer):
- Figure B (ratio vs step):
- Figure C (layer × step heatmap):

---

## Alpha oracle (primary table)

Routing outside middle band: \(s \ge 0.995\) → reuse; \(s < 0.99\) → full.

| Middle band | Acc (flexible) | cos(h_partial, h_full) | KL(p_full ‖ p_partial) | top-1 agree | first divergence step |
|-------------|---------------:|-----------------------:|-----------------------:|------------:|----------------------:|
| reuse (α=0) | 77.10 | | | | |
| α=0.25 | | | | | |
| α=0.50 | | | | | |
| α=0.75 | | | | | |
| full (α=1) | 78.39 | | | | |

Minimal effective α (recovers most of full):

-

---

## Paired answer flips

| Base (reuse α=0) | Partial | Count |
|------------------|---------|------:|
| wrong | correct | |
| correct | wrong | |
| correct | correct | |
| wrong | wrong | |

Hand-checked **wrong → correct** cases: evidence of necessary delta recovery? yes / no / mixed

Notes:

-

---

## SVD (only if alpha is positive)

| r/d | Top-r Acc | Random-r Acc | Bottom-r Acc | Top-r KL | Random-r KL |
|----:|----------:|-------------:|-------------:|---------:|------------:|
| 1/64 | | | | | |
| 1/32 | | | | | |
| 1/16 | | | | | |
| 1/8 | | | | | |
| 1/4 | | | | | |
| 1/2 | | | | | |

Ordering holds on KL / decoding / acc?

> Top-r > Random-r > Bottom-r ? yes / no

Key-delta claim supported? yes / no

---

## Layer / block analysis

| Question | Result |
|----------|--------|
| Effective rank layer-dependent? | yes / no |
| \(r_{90}\) range across layers | |
| Adjacent-layer subspace similarity high for 2–4 layers? | yes / no |
| Prefer layer-wise or block-wise shared basis? | layer / block / unclear |

---

## Core answers (checklist)

| Question | Result |
|----------|--------|
| middle band 占比 | |
| reuse acc | 77.10 (5090 ref) |
| 最小有效 α | |
| partial 能否恢复 full | yes / no |
| top-r 是否优于 random-r | yes / no / N/A |
| effective rank 是否 layer-dependent | yes / no / N/A |
| block-wise 是否有迹象 | yes / no / N/A |

---

## One-line conclusion

> Middle-band partial is **worth pursuing** / **not worth pursuing** / **inconclusive**.

Detail:

-
