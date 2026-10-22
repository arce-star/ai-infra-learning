# AI Infra Learning

A hands-on learning repository for LLM inference and AI infrastructure.

The goal is not to memorize isolated terms. The goal is to understand how one LLM request travels through a real system, reproduce the important mechanisms in small experiments, and gradually build enough background to read and contribute to systems such as vLLM, SGLang and DeepInfer.

## Main learning path

```text
Run one model
    ↓
Serve an OpenAI-compatible API
    ↓
Measure latency / throughput / memory
    ↓
Understand batching / scheduling / KV cache
    ↓
Add observability
    ↓
Study distributed serving and parallelism
    ↓
Read DeepInfer with system context
```

## Repository structure

```text
docs/
  00-ai-infra-map.md       System map and learning order
  01-glossary.md           Terms, definitions and links between concepts
labs/
  01-vllm-server/          First runnable inference service
notes/
  learning-log.md          What I learned, what confused me, what changed
```

More labs will be added only after the previous concepts are understood experimentally.

## Current phase

**Phase 1 — Single-GPU LLM serving**

Current objective:

> Run a small model with vLLM, expose an OpenAI-compatible API, send requests to it, and observe GPU memory and latency.

Do not start with DeepInfer internals yet. DeepInfer currently targets production-scale inference and exposes concepts such as asynchronous scheduling, Attention DP / Context Parallelism, MoE Expert Parallelism, EPLB and optimized kernels. These become much easier to understand after building a small serving system first.

## Learning rule

For every new term, answer five questions:

1. What problem does it solve?
2. Where does it sit in the request path?
3. What metric does it affect?
4. Can I observe it in a small experiment?
5. Where does the same concept appear in vLLM / DeepInfer?

## References

- USTC LLM platform: https://llm.ustc.edu.cn/
- vLLM: https://github.com/vllm-project/vllm
- DeepInfer: https://github.com/deepseek-ai/DeepInfer
- SGLang: https://github.com/sgl-project/sglang

## Security note

Never commit API keys, internal server addresses, credentials, private USTC configuration, or non-public operational data to this repository.
