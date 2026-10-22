# AI Infra Glossary

This glossary is organized by system role, not alphabetically.

## Request and generation

| Term | Short meaning | Why it matters |
|---|---|---|
| Token | Basic text unit processed by the model | Compute and context length are measured in tokens |
| Tokenizer | Converts text to/from token IDs | Determines token count and chat formatting |
| Context length | Maximum sequence length considered by the model | Longer context increases memory and prefill cost |
| Prefill | Process the input prompt and build attention state | Dominates work before the first output token |
| Decode | Generate tokens autoregressively after prefill | Repeats once per generated step |
| Sampling | Select next token from model probabilities | Controls output behavior; also part of runtime |

## Performance

| Term | Short meaning | Usually affected by |
|---|---|---|
| TTFT | Time To First Token | queueing, prefill, scheduling |
| TPOT | Time Per Output Token | decode efficiency |
| TBT | Time Between Tokens | streaming decode performance |
| Latency | Time for a request to complete | all stages |
| Throughput | Total useful work per unit time | batching, utilization, kernels |
| QPS | Queries/requests per second | concurrency + service capacity |
| Concurrency | Simultaneously active requests | scheduler, batching, memory |

## Memory and cache

| Term | Short meaning | Why it matters |
|---|---|---|
| HBM | GPU high-bandwidth memory | Holds weights, KV cache and runtime tensors |
| KV Cache | Cached attention keys/values for previous tokens | Avoids recomputing prior attention state during decode |
| Prefix Cache | Reuse KV state for repeated prompt prefixes | Can reduce repeated prefill work |

## Scheduling

| Term | Short meaning | Why it matters |
|---|---|---|
| Batch | Multiple sequences processed together | Improves hardware utilization |
| Continuous Batching | Add/remove requests while generation continues | Better serving throughput than fixed batches |
| Scheduler | Chooses which requests/tokens execute next | Central to latency/throughput tradeoffs |
| Chunked Prefill | Split long prefill into smaller chunks | Prevents one long prompt from monopolizing execution |

## Parallelism

| Term | Meaning | Typical use |
|---|---|---|
| DP | Data Parallelism | replicate model/service across workers |
| TP | Tensor Parallelism | split tensor operations across GPUs |
| PP | Pipeline Parallelism | split model layers into stages |
| CP | Context Parallelism | split sequence/context computation |
| EP | Expert Parallelism | distribute MoE experts |
| EPLB | Expert Parallel Load Balancing | balance MoE expert load across devices |

## Communication

| Term | Short meaning |
|---|---|
| NCCL | NVIDIA collective communication library |
| AllReduce | Aggregate values across workers and distribute result |
| AllToAll | Each worker exchanges different data with every other worker; common in MoE |
| NVLink | High-bandwidth GPU-to-GPU interconnect |
| RDMA | Remote memory access with low CPU overhead / latency |

## Serving and production

| Term | Short meaning |
|---|---|
| API Gateway | Front door for requests, auth, quotas, routing policies |
| Router | Chooses backend/replica for a request |
| Load Balancer | Distributes traffic across serving instances |
| Replica | Another service/model instance capable of handling requests |
| Metrics | Numeric measurements over time |
| Logs | Event records |
| Traces | End-to-end timing/path of an individual request |
| OpenTelemetry | Standard instrumentation ecosystem for metrics/logs/traces |

## GPU execution

| Term | Short meaning |
|---|---|
| CUDA | NVIDIA GPU programming/runtime platform |
| Kernel | Function executed on the GPU |
| GEMM | Matrix multiplication primitive central to neural networks |
| CUDA Graph | Capture/replay GPU work to reduce launch overhead |
| FlashAttention | IO-aware optimized attention implementation |

## Rule for adding a new term

Do not add only a dictionary definition. Add these answers when the term becomes important:

```text
Term:
System layer:
Problem it solves:
Metric it affects:
Related terms:
Experiment where I observed it:
vLLM/DeepInfer location (later):
```
