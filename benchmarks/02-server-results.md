# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=5` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 85 | 1.44 | 5900 | 8200 | 8900 | 8.3 | 0.0% |
| 50 | 81 | 1.37 | 29000 | 38000 | 40000 | 34.0 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.95x** (19% of linear) |
| P95 latency | **4.63x** |
| Effective concurrency at 50 users | 34.0 vs `--parallel 4` slots (occupancy/slot ratio 8.49) |

**Saturated.** Throughput delivered only 0.95x for 5x the offered load, and effective concurrency (34.0) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.95x while P95 moved 4.63x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading
The server was at or beyond capacity by 10 users: effective concurrency was 8.3
against 4 slots. From 10 to 50 users, throughput fell from 1.44 to 1.37 RPS
(0.95x) while P95 rose from 8.2 to 38 s (4.63x). The concurrent metrics capture
showed 3.98 busy slots and 46 deferred requests. I would first test a lower
output-token limit to shorten decode and queue time, accepting shorter answers;
then sweep `--parallel` if the P95 SLO still fails.
