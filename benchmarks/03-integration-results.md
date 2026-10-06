# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 1168.3 | 1168.3 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 750.4 | 750.4 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 658.9 | 659.0 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **859.2** · total **859.2**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real
N16 cloud/IaC is stubbed (localhost), N17 data pipeline is stubbed (no pipeline
job), N18 lakehouse is stubbed (toy in-memory documents), and N19 vector/features
is stubbed (keyword-overlap retrieval). N20 serving is real. LLM serving averaged
859.2 ms and accounted for essentially all measured latency; embed and retrieval
were 0 ms in this keyword-only path. To halve total latency, I would first reduce
LLM generation work, for example by limiting output length, then measure again.
