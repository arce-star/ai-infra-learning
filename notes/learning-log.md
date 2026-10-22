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
12. The first `/v1/chat/completions` request succeeded. Qwen3 defaulted to thinking mode, emitted a `<think>` block, used all 128 completion tokens, and returned `finish_reason=length` before reaching the final answer. For clean serving tests, disable thinking with `chat_template_kwargs: {"enable_thinking": false}`.
13. With thinking disabled, the second request completed normally with `finish_reason=stop`, but the 0.6B model still gave an incorrect explanation of inference decode. This separates serving correctness from model-quality correctness: the API/engine can work perfectly while the model output is semantically wrong.
14. The first streaming benchmark measured TTFT and TPOT. TPOT stayed very stable across prompt lengths, around 2.45–2.67 ms/token (about 375–408 output tok/s), which is a clean decode-speed signal.
15. The first TTFT sweep is not yet a clean prompt-length comparison. The short case was the first streaming request and paid extra first-use overhead, while medium and long share a large prefix and prefix caching is enabled. The benchmark should be repeated with warmup, unique prefixes, multiple repetitions and medians.

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
First /v1/chat/completions request: success
First request usage: 23 prompt tokens + 128 completion tokens = 151 total tokens
First request finish_reason: length
First request note: Qwen3 default thinking mode consumed the 128-token budget before producing the final answer
Second request with thinking disabled: success
Second request usage: 27 prompt tokens + 38 completion tokens = 65 total tokens
Second request finish_reason: stop
Second request note: serving succeeded, but the 0.6B model still explained decode incorrectly

First streaming benchmark:
short:  18 prompt tokens, TTFT 100.28 ms, TPOT 2.67 ms/token, 374.42 tok/s
medium: 342 prompt tokens, TTFT 29.85 ms, TPOT 2.45 ms/token, 407.59 tok/s
long: 1622 prompt tokens, TTFT 63.98 ms, TPOT 2.63 ms/token, 380.67 tok/s

Interpretation:
- TPOT is already stable and informative.
- TTFT is confounded by first-use overhead and prefix-cache reuse, so do not interpret the short/medium/long ordering literally yet.
- Total latency is approximately TTFT + decode duration. For example, the short case is about 100.28 ms + 63 × 2.67 ms ≈ 268.5 ms, close to the observed 269.19 ms.

Anything surprising: checkpoint size is much smaller than serving VRAM usage because vLLM reserves substantial memory for KV cache and runtime state. Serving correctness and model-answer correctness are separate concerns.
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

Repeat TTFT with a cleaner methodology: one warmup request, unique prefixes to defeat prefix-cache reuse, several repetitions per prompt length, and median reporting. Then benchmark concurrency.
