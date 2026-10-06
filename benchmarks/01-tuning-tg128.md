# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **10 physical · 10 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 52.1 | 97% |
| 5 | 53.7 | 100% |
| 10 | 49.6 | 92% |
| 20 | 48.3 | 90% |

**Best**: `-t 5` at 53.7 tok/s
**Slowest tested**: `-t 20` at 48.3 tok/s (1.11x spread)
**Against the physical-core default** (`-t 10`, 49.6 tok/s): 1.08x

Use this in your run:

```bash
LAB_N_THREADS=5 make bench
```

## Your explanation

The best tested point was 5 threads at 53.7 tok/s. Throughput was already close
at 1 thread (52.1 tok/s), then fell to 49.6 tok/s at 10 threads and 48.3 tok/s
at 20; using 5 instead of the 10-thread default improved this benchmark by 1.08x.
The knee is therefore at or below 5 threads in this sweep. Since `ngl=99` on the
M1 Pro, most model layers are offloaded to Metal, so CPU thread count is not a
simple proxy for decode parallelism. The small decline at higher counts is
consistent with added CPU scheduling/coordination overhead or contention; this
sweep alone does not identify which one. The result supports using 5 threads for
the next measurement, while keeping the finding specific to this benchmark setup.
