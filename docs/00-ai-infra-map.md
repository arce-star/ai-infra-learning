# AI Infra System Map

This page is the mental map for the repository. New terms should be placed somewhere on this map instead of memorized independently.

## 1. One request through the system

```text
User / Application
      ↓
API Gateway / Authentication / Rate Limit
      ↓
Router / Load Balancer
      ↓
Inference Engine
      ↓
Tokenizer → Scheduler → Prefill → KV Cache → Decode → Sampling
      ↓
Model Runtime
      ↓
PyTorch / CUDA / Kernels
      ↓
GPU memory + compute
      ↓
NVLink / PCIe / RDMA / network when distributed
```

Observability sits across the whole path:

```text
Metrics + Logs + Traces
Prometheus / Grafana / OpenTelemetry
```

## 2. The five layers

### Layer A — Model and request

Learn first:

- Token / tokenizer
- Context length
- Prompt tokens / output tokens
- Prefill
- Decode
- Sampling

Key question: **What computation happens before the first token, and what happens for every later token?**

### Layer B — Inference engine

Examples: vLLM, SGLang, DeepInfer, TensorRT-LLM.

Learn:

- Scheduler
- Batch
- Continuous batching
- Chunked prefill
- KV cache
- Prefix cache
- Speculative decoding

Key question: **How does the engine keep the GPU busy while many requests have different lengths?**

### Layer C — GPU execution

Learn later:

- CUDA
- Kernel
- GEMM
- FlashAttention / FlashMLA
- CUDA Graph
- HBM

Key question: **Where are compute time and memory bandwidth actually spent?**

### Layer D — Distributed inference

Learn after single-GPU serving is clear:

- DP: Data Parallelism
- TP: Tensor Parallelism
- PP: Pipeline Parallelism
- CP: Context Parallelism
- EP: Expert Parallelism
- NCCL
- AllReduce / AllToAll
- NVLink / RDMA
- Prefill-Decode disaggregation

Key question: **Why does one model need multiple devices, and what communication does each strategy introduce?**

### Layer E — Production serving

Learn after benchmarking:

- API Gateway
- Router
- Load Balancer
- Replica
- Autoscaling
- Fault tolerance
- Rate limiting
- Metrics / Logs / Traces
- Prometheus / Grafana / OpenTelemetry

Key question: **How do we turn an inference engine into a reliable service for many users?**

## 3. Performance metrics

```text
Request arrives
     |
     |---- TTFT ----> first output token
     |                 |
     |                 |-- TPOT/TBT -- token -- token -- ...
     |
     |---------------- E2E Latency ------------------------->
```

- **TTFT**: Time To First Token. Strongly affected by queueing and prefill.
- **TPOT/TBT**: Time Per Output Token / Time Between Tokens. Closely related to decode performance.
- **Throughput**: amount of useful work completed per unit time, often tokens/s.
- **QPS**: requests completed per second.
- **Concurrency**: number of active requests.
- **GPU memory**: model weights + KV cache + temporary/runtime memory.

Latency and throughput are often a tradeoff. A production system rarely optimizes only one number.

## 4. Why start from vLLM before DeepInfer?

DeepInfer exposes production-scale mechanisms such as asynchronous scheduling, Attention DP / balanced CP, MoE EP + EPLB, speculative decoding and optimized kernels. Its open-source release currently focuses on core inference rather than a complete online service.

The shortest path to understanding those mechanisms is:

```text
vLLM single GPU
→ observe scheduler / batching / KV cache
→ benchmark
→ multi-GPU concepts
→ DeepInfer architecture and source code
```

## 5. Current checkpoint

The current lab should make the following words concrete rather than theoretical:

- model server
- OpenAI-compatible API
- request
- prompt tokens
- output tokens
- TTFT
- GPU memory
- concurrency

Do not move on until a request can be sent to a model server and you can explain, approximately, what happened between request arrival and the first generated token.
