# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Linux-x86_64` · llama.cpp `b10488`
CPU: **4 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 29.6 | 100% |
| 2 | 29.0 | 98% |
| 4 | 28.1 | 95% |
| 8 | 27.2 | 92% |
| 16 | 26.6 | 90% |

**Best**: `-t 1` at 29.6 tok/s
**Slowest tested**: `-t 16` at 26.6 tok/s (1.11x spread)
**Against the physical-core default** (`-t 4`, 28.1 tok/s): 1.05x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## Your explanation (required -- replace this line)

Tôi nghĩ nguyên nhân mà mô hình lại chạy tốt nhất ở 1 luòng là vì: khi chạy trên GPU, cho nên tốc độ decode bị giới hạn bởi GPU và băng thông RAM của GPU. Các nhân CPu khi đó chỉ tăng scheduling overhead, và áp lực lên cache/memory bandwidth của CPU khi điều phối lệnh GPU.
---
