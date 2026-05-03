# Self-Hosted LLM Serving: A Reference Architecture

> An opinionated, documented reference for self-hosting open-source LLMs at small-to-medium scale. Built as a learning artifact, not a product.

**Status:** 🌱 Early-stage. Started May 2026. The repository is intentionally public from day one as a working lab notebook - expect rough edges, in-progress sections, and visible iteration. See [Roadmap](#roadmap) for current focus.

---

## What this is

A study and reference deployment for organisations and engineers who want to **self-host open-source LLMs** (e.g. Llama, Qwen, Mistral) for **10–1,000 concurrent users** on a **single node or small multi-node setup**.

It is intended to answer questions like:

- What does a *credible* production-grade self-hosted LLM stack look like end-to-end?
- What are the real tradeoffs between serving engines, quantization formats, and hardware choices?
- How does a self-hosted setup actually behave under realistic enterprise load — latency percentiles, throughput, cost per million tokens?
- Where do the sharp edges live, and how do you observe them in production?

Each architectural choice is documented as an [Architecture Decision Record (ADR)](./docs/adr/) with the alternatives considered and the reasons for the decision.

## What this is *not*

To keep scope honest:

- **Not a product or platform.** If you need a managed solution, look at BentoML, Anyscale, or Modal.
- **Not for foundation-model-scale workloads.** This targets small-to-medium deployments, not multi-thousand-GPU serving.
- **Not for hobbyist single-user setups.** For that, `ollama` or `llama.cpp` directly is simpler and better.
- **Not a benchmark of model quality.** This is about *serving infrastructure*, not which model is "best."
- **Not production-ready out of the box.** It's a reference to learn from and adapt, not a turn-key deployment.

## Why this exists

Most existing material on self-hosted LLM serving falls into two camps: toy examples (a single `ollama serve` command with no surrounding infrastructure), or proprietary internal documentation that nobody outside the company sees. This repository tries to fill the gap in between — a credible, opinionated, fully documented reference deployment that an engineer can study, fork, and adapt.

It's also a personal learning project. I'm using this to go deep on LLM inference systems. The writeups document what I learned, not just what I built.

## Architecture (current)

> This section will evolve. Current high-level shape:

```
[Clients] → [Gateway / Auth] → [Router] → [vLLM Workers] → [Models on GPU]
                                  │
                                  ├─ [Prometheus / Grafana]   (metrics)
                                  ├─ [OpenTelemetry]          (traces)
                                  └─ [Eval & Load Test Harness]
```

Components and rationale live in [`docs/architecture.md`](./docs/architecture.md). Decisions live in [`docs/adr/`](./docs/adr/).

## Repository structure

```
.
├── README.md                  # You are here
├── docs/
│   ├── architecture.md        # High-level architecture & current state
│   ├── adr/                   # Architecture Decision Records
│   └── writeups/              # Long-form posts on what I learned
├── deploy/                    # Kubernetes / Docker Compose manifests
├── serving/                   # vLLM configuration & wrappers
├── observability/             # Prometheus, Grafana, OpenTelemetry config
├── loadtest/                  # Load testing harness & scenarios
├── evals/                     # Quality evaluation under load
└── scripts/                   # Reproducible setup & teardown
```

(Directories appear as the work progresses — empty ones are listed in the roadmap below.)

## Getting started

> ⚠️ Not yet runnable end-to-end. This section will be filled in as components land.

The eventual target: a `make up` (or equivalent) that brings up a working stack on either a single GPU node or a small Kubernetes cluster, with documented requirements and cost expectations.

## Roadmap

I'm tracking work as GitHub issues and milestones. The rough sequence:

- [ ] **Phase 1 — Single-node vLLM serving** with sane defaults, basic Docker setup
- [ ] **Phase 2 — Observability layer** (Prometheus, Grafana, OpenTelemetry, vLLM metrics)
- [ ] **Phase 3 — Load testing harness** with realistic enterprise traffic patterns
- [ ] **Phase 4 — Quality-under-load evaluation** harness
- [ ] **Phase 5 — Architecture decision documentation** caught up with reality
- [ ] **Phase 6 — Cost & throughput study** as the first long-form writeup
- [ ] **Phase 7 — Multi-tenant / multi-model serving** experiments
- [ ] **Phase 8 — Quantization study** (FP16 vs INT8 vs INT4, GPTQ vs AWQ)

Phases will overlap. Issues will track concrete work items.

## Writeups

Long-form posts on what I learned will appear in [`docs/writeups/`](./docs/writeups/) and be cross-posted to my blog. Planned early posts:

- *"Choosing a serving engine in 2026: vLLM, TensorRT-LLM, SGLang"*

## Design principles

A few rules I'm trying to hold to:

1. **Honest tradeoffs over claimed perfection.** Every decision document calls out what the choice gives up.
2. **Reproducibility is part of the artifact.** If you can't run it, it doesn't exist.
3. **Negative and surprising results are first-class.** The benchmark that disproved my assumption is more valuable than the one that confirmed it.
4. **Scope discipline.** Better to do a small thing well than a large thing badly.
5. **Writing is half the work.** Code without explanation is half-shipped.

## Hardware & cost expectations

Most experimentation is done on rented cloud GPUs (Vast.ai / Lambda / RunPod) — typically a single A100 for short-lived experiments. The repository is explicit about cost in any benchmark or study.

## Contributing

This is a personal learning project, so I'm not actively seeking contributions. That said:

- **Issues are welcome** — especially "I tried this and it broke" or "your ADR missed an alternative."
- **Corrections are very welcome.** I'd rather be told I'm wrong than stay wrong.
- **PRs**: please open an issue first to discuss.

## License

MIT — see [LICENSE](./LICENSE).

---

*This README is itself a living document. Last updated: 2026-05-03.*
