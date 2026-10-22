# Learning Log

Use this file for short observations, not polished notes.

## 2026-10-23 — Project start

### What I already know roughly

- Basic Linux / command line usage
- GPU and CUDA at a user level
- PyTorch experiments
- Calling APIs and using AI agents
- First-pass definitions of common AI Infra terms

### Current target

Build a minimal local LLM serving system with vLLM and make the request path concrete.

### Environment setup lessons

1. GPU memory usage and GPU compute utilization are different resources. A workload can use only a small amount of VRAM while still keeping GPU compute near 100% busy.
2. Python package installation depends on the configured package index/mirror. AutoDL's default Aliyun mirror did not provide `uv`, while installing from PyPI worked.
3. A Python package being installed successfully does not mean its native binaries are compatible with the host GPU software stack.
4. The initial `vllm==0.32.0` PyPI install pulled the default CUDA 13 build. The host uses NVIDIA driver 570.124.04 and reports CUDA 12.8 support, so the vLLM CLI failed with `ImportError: libcudart.so.13`.
5. The next fix is to install the official CUDA 12.9 vLLM wheel instead of trying to force the CUDA 13 build onto the host.

### Lab 01 observations

Fill this after running the first server:

```text
GPU: RTX 3090 24 GB
Model:
GPU memory after model load:
Rough model loading time:
First request result:
Short-prompt TTFT feeling:
Long-prompt TTFT feeling:
Anything surprising:
```

### Explain in my own words

**Model checkpoint:**

**Inference engine:**

**API server:**

**Prefill:**

**Decode:**

**KV cache:**

### Questions I still do not understand

- 

### Next step

Fix the vLLM CUDA build, start the first server, then write a real benchmark client for TTFT / latency / throughput / concurrency.
