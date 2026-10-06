# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=5` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 3062 | 108 / 120 | 18.6 / 21.2 | 1275 / 1446 / 1446 | 53.7 |
| UD-Q2_K_XL | 2.24 | 3056 | 108 / 112 | 17.5 / 17.9 | 1205 / 1233 / 1233 | 57.1 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.06x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation
For this 5-thread run, Q2 was 1.06x faster by decode rate (57.1 vs 53.7 tok/s)
and 0.73 GB smaller (about 25%). On the same short quality prompt, both variants
recommended 4-bit for better answer quality; their brief answers agreed, so this
single prompt showed no clear quality difference. I would use Q4 when answer
quality is the priority and Q2 when the smaller footprint or decode speed matters;
one prompt is too little to claim the two are equivalent overall.
