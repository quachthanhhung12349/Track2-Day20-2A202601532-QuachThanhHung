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

_Where is the knee, and why there? If the peak sits at your physical core count
and drops above it, say what the extra threads are competing for. If your curve
does something else -- flat, or still climbing at 2x logical cores -- say that
instead and reason about why. A result that contradicts the expected shape is
worth more than one that matches it, as long as you explain it._
