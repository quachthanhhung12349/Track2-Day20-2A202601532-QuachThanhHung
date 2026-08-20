# 02 - Serve: load test + saturation reading

Host `Linux-x86_64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=4` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 13 | 0.23 | 31000 | 55000 | 55000 | 8.0 | 0.0% |
| 50 | 39 | 0.69 | 31000 | 56000 | 56000 | 20.3 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **3.01x** (60% of linear) |
| P95 latency | **1.02x** |
| Effective concurrency at 50 users | 20.3 vs `--parallel 4` slots (occupancy/slot ratio 5.08) |

**At capacity, still scaling.** All 4 decode slots are busy (effective concurrency 20.3) but throughput still rose 3.01x. You are at the knee -- the next increment of load is where P95 starts to run away.

P95 grew no faster than throughput (1.02x vs 3.01x), so this server still has headroom at 50 users.

> **Small sample.** Only 13 requests completed in the
> shorter run, so these percentiles are indicative rather than solid. Note also that
> locust averages only *completed* requests: when the run ends with requests still
> queued, effective concurrency is an **under**-estimate. Trust the throughput-scaling
> row over the concurrency row here, and run longer (`-t 3m`) if you want firmer numbers.

## Your reading (required -- replace this line)

_Where does your server saturate, and what is the evidence? Name the number that
convinced you. Then say what you would change first to raise goodput at your SLO --
and why that knob and not another._
