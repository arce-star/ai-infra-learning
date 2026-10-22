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
5. Installing the official `vllm==0.32.0+cu129` wheel fixed the vLLM CUDA build, but `torchcodec==0.17.0` from PyPI still expected CUDA 13 (`libnvrtc.so.13`).
6. Replacing it with `torchcodec==0.16.0+cu129` fixed the remaining binary mismatch.
7. Final working environment: Python 3.12.3, PyTorch 2.13.0+cu129, vLLM 0.32.0+cu129, torchcodec 0.16.0+cu129 on an RTX 3090 (compute capability 8.6). A CUDA matrix multiply test succeeds and the vLLM CLI starts normally.
8. Multi-index dependency resolution matters. `uv --index-strategy unsafe-best-match` was needed because the PyTorch CUDA index contained an older `requests` package that otherwise blocked dependency solving.

### Lab 01 observations

```text
GPU: RTX 3090 24 GB
Model: Qwen/Qwen3-0.6B (planned)
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

Download the Qwen3-0.6B model explicitly to the AutoDL data disk, inspect the checkpoint files, then load that local checkpoint with vLLM and make the first OpenAI-compatible API request.
