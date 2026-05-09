# ADR-001: Use vLLM as the LLM Serving Engine

- **Date:** 2026-05-09
- **Status:** Proposed
- **Deciders:** @Pewtas
- **Supersedes:** —
- **Superseded by:** —

---

## Context

OpenServe needs an inference engine to serve open-source LLMs
(Llama, Qwen, Mistral) to internal users. The engine sits behind
LiteLLM Proxy (ADR-002) and is never called directly by clients.

Constraints:

- Target scale: 10–1,000 concurrent users; organisation of ~10,000 employees
  with realistic peak concurrency of 5–15% (~500–1,500 simultaneous requests)
  This "metric" is extremly volatile and heavly dependent on the sector and
  the "ai-readiness" of the company
- Must expose an OpenAI-compatible `/v1/chat/completions` API
- Must run on a single GPU node initially (A100 80GB); multi-node later
- Fully open-source and self-hostable — no proprietary cloud dependency
- Operator is a single engineer; operational complexity is a real cost
- Reference architecture goal: reproducibility and documentation matter as
  much as raw performance

---

## Decision

Use **vLLM** as the LLM serving engine.

```bash
docker run --runtime nvidia --gpus all \
  -p 8000:8000 \
  -v /models:/models \
  vllm/vllm-openai:latest \
  --model /models/meta-llama/Llama-3.1-8B-Instruct \
  --served-model-name llama-3.1-8b \
  --max-model-len 8192 \
  --tensor-parallel-size 1 \
  --enable-chunked-prefill \
  --gpu-memory-utilization 0.90
```

At target scale on a single A100 80GB, vLLM can sustain 50–150 simultaneous
streaming requests (model-dependent) before TTFT degrades materially.
For 1,000 active users this is sufficient only under realistic enterprise
usage patterns — average think/read time of ~50s against ~10s generation
time yields ~170 requests in flight at peak, which sits within the A100's
comfortable operating range for an 8B model. A 70B model on the same hardware
pushes this to the limit; two GPUs or a smaller model is the mitigation.
Workload profiling in Phase 3 will validate or challenge this assumption.

---

## Alternatives Considered

| Option | Why rejected |
|---|---|
| **SGLang** | higher throughput via RadixAttention, but only on shared-prefix workloads (multi-turn, RAG). At target scale vLLM is not the bottleneck, so the gain is irrelevant. Smaller ecosystem, fewer integrations, less community support. Legitimate future consideration if workload profiling shows heavy prefix reuse. |
| **TensorRT-LLM** | Highest raw performance, but requires a long engine compilation per model. Every model swap or update triggers a recompile. Eliminates fast iteration, which is incompatible with a reference architecture goal. NVIDIA-only — no AMD or alternative hardware support. Operational overhead unjustified at this scale. |
| **Ollama** | Appropriate for single-user or developer setups. Not designed for multi-user concurrent serving; lacks the scheduling and batching primitives needed for 500+ concurrent requests. Used locally on Mac for development (see ADR-002 on dev/prod parity). |
| **llama.cpp server** | CPU-first, good for edge/embedded. GPU throughput significantly below vLLM. No production batching at target scale. |
| **Do nothing / use cloud API** | Eliminates data sovereignty. Ongoing cost at 10,000-employee scale is prohibitive. Contradicts the core self-hosted premise. |

---

## Consequences

**Positive:**

- PagedAttention eliminates KV-cache fragmentation; GPU memory utilisation
  stays high under concurrent load without manual tuning
- Continuous batching means new requests join in-flight batches immediately —
  no fixed batch windows, predictable latency under load
- OpenAI-compatible API out of the box; LiteLLM, LlamaIndex, and Haystack
  all treat vLLM as a first-class backend
- Broadest model support in the ecosystem — Llama, Qwen, Mistral, Mixtral,
  Phi, Gemma all work without custom code
- Hardware flexibility: NVIDIA today, AMD ROCm and AWS Trainium supported
  if hardware strategy changes
- 1,000+ contributors; GitHub issues contain answers to most production
  problems; documentation is the most complete of any OSS serving engine
- ~62s cold start — fast enough for model rotation without operational pain

**Negative / risks:**

- p99 TTFT latency shows higher variance than SGLang and TensorRT-LLM under
  extreme load; not a concern at target scale but relevant if concurrency
  grows beyond ~500 simultaneous requests
- vLLM releases frequently and occasionally introduces breaking changes
  in minor versions — pin the Docker image tag, never use `:latest` in
  production
- If the workload profile shifts heavily toward RAG or multi-turn agents
  with shared system prompts, SGLang's RadixAttention advantage becomes
  material; revisit this decision at that point
- Memory overhead is higher than llama.cpp for small models on limited
  VRAM — not relevant for A100/H100 targets

---

## Links

- [vLLM docs](https://docs.vllm.ai)
- [vLLM GitHub](https://github.com/vllm-project/vllm)
- [Spheron H100 benchmark (2026)](https://www.spheron.network/blog/vllm-vs-tensorrt-llm-vs-sglang-benchmarks/)
- Related: ADR-002 (API gateway: LiteLLM), ADR-003 (Observability stack)