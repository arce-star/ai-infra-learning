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
9. Hugging Face Hub was unreachable from the AutoDL instance, so the same official Qwen checkpoint was downloaded from ModelScope instead. Model source and inference engine are separate concerns once the local checkpoint is complete.
10. The first vLLM startup attempt reached model loading, torch.compile, CUDA graph capture and KV-cache allocation, then failed because FlashInfer's JIT sampler called the system `/usr/local/cuda/bin/nvcc` with an unsupported `--compress-mode=size` flag. Disabling the FlashInfer sampler allowed the server to start.
11. `nvidia-smi` showing a GPU process does not prove the HTTP server is ready. A successful `/v1/models` response is a direct readiness signal. The container did not have the `ss` command installed, so endpoint checks were more useful than relying on port-inspection tooling.

### Model checkpoint downloaded

Local path:

`/root/autodl-tmp/ai-infra-lab/models/Qwen3-0.6B`

Observed size: about 1.5 GB.

Important files:

- `model.safetensors`: model parameters / weights
- `config.json`: model architecture/configuration
- `generation_config.json`: generation defaults
- `tokenizer.json`, `tokenizer_config.json`, `vocab.json`, `merges.txt`: tokenizer assets

### Lab 01 observations

```text
GPU: RTX 3090 24 GB
Model: Qwen3-0.6B
Checkpoint size: ~1.5 GB
GPU memory after model load/server startup: ~12.2 GiB
vLLM setting: --gpu-memory-utilization 0.50
Observed EngineCore process: ~12.2 GiB VRAM
Model loading memory reported by vLLM: ~1.12 GiB
CUDA graph memory reported by vLLM: ~0.30 GiB
Available KV cache memory reported by vLLM: ~9.71 GiB
GPU KV cache capacity: 90,928 tokens
Max concurrency estimate at 4096 tokens/request: 22.20x
/v1/models: success
First /v1/chat/completions request: pending
Short-prompt TTFT feeling:
Long-prompt TTFT feeling:
Anything surprising: checkpoint size is much smaller than serving VRAM usage because vLLM reserves substantial memory for KV cache and runtime state.
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

Send the first `/v1/chat/completions` request, then compare short / medium / long prompts and explain the serving path in my own words.
