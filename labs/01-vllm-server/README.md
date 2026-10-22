# Lab 01 — Run a vLLM Model Server

## Goal

Turn a model into a network service and successfully call it through an OpenAI-compatible API.

By the end of this lab, you should be able to explain the difference between:

- a model checkpoint
- an inference engine
- an API server
- a client request
- prefill and decode

## Recommended environment

Use a separate Linux GPU environment. Do not install this into an existing research environment that you care about.

The official vLLM quickstart currently recommends Python 3.12 with `uv` for NVIDIA GPUs.

## Step 1 — Create a clean environment

```bash
cd /root/autodl-tmp
mkdir -p ai-infra-lab
cd ai-infra-lab

python -m pip install -U uv
uv venv --python 3.12 --seed
source .venv/bin/activate
uv pip install vllm --torch-backend=auto

python -V
vllm --help | head -40
nvidia-smi
```

## Step 2 — Start a small model server

Use a small model first so the experiment is about serving rather than model size.

```bash
source /root/autodl-tmp/ai-infra-lab/.venv/bin/activate

vllm serve Qwen/Qwen3-0.6B \
  --host 0.0.0.0 \
  --port 8000 \
  --generation-config vllm
```

Keep this terminal running.

While the model is loaded, note the output of:

```bash
nvidia-smi
```

Record approximately how much GPU memory the process occupies.

## Step 3 — Verify the server from another terminal

```bash
curl http://127.0.0.1:8000/v1/models
```

Then send a chat request:

```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen3-0.6B",
    "messages": [
      {"role": "system", "content": "You are a concise assistant."},
      {"role": "user", "content": "Explain the difference between prefill and decode in LLM inference in two sentences."}
    ],
    "temperature": 0,
    "max_tokens": 128
  }'
```

## Step 4 — Observe rather than optimize

For the first run, record only:

```text
GPU model:
Model:
Model loading time (rough):
GPU memory after loading:
Prompt:
Prompt token count (if shown):
Output token count (if shown):
Approximate time until first visible output:
Approximate total response time:
```

Then answer in your own words:

1. What is vLLM doing that the Qwen checkpoint itself does not do?
2. What does `localhost:8000` represent?
3. What happened before the first output token appeared?
4. What happens repeatedly after the first output token?
5. Why does the server still occupy GPU memory when no request is being processed?

## Step 5 — First small experiment

Send the same request with three prompt lengths:

- short: ~20 tokens
- medium: ~500 tokens
- long: ~2000 tokens

Do not benchmark rigorously yet. Just observe whether time-to-first-token changes more obviously than token-generation speed.

Write your observation in `notes/learning-log.md`.

## Completion criterion

This lab is complete when you can draw this from memory and explain every arrow:

```text
curl/client
    ↓ HTTP request
vLLM API server
    ↓
scheduler
    ↓
prefill
    ↓
KV cache
    ↓
decode → decode → decode
    ↓
stream/final response
```

## Next lab

Only after this works, build a small benchmark client that measures TTFT, end-to-end latency and throughput under different concurrency levels.
