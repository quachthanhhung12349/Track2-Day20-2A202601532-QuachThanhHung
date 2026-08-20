# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Linux-x86_64` · llama.cpp `b10488`
Settings: `threads=4` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 10375 | 380 / 1642 | 33.8 / 34.1 | 2513 / 3761 / 3761 | 29.6 |
| UD-Q2_K_XL | 2.24 | 5074 | 354 / 2152 | 43.2 / 46.1 | 3066 / 4866 / 4866 | 23.1 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.28x SLOWER** than `UD-Q4_K_XL` here, despite being 0.73 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Your observation (required -- replace this line)

2 bit nhanh hơn 4 bit khoảng 2 lần khi load model, tuy nhiên khi decode model thì model 4 bit lại cho tốc độ nhanh hơn. Vì máy tôi đủ VRAM -> không nên chạy model 2 bit ở đây.
