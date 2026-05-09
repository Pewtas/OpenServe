# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A documented reference deployment for self-hosting open-source LLMs (Llama, Qwen, Mistral) at **10–1,000 concurrent users** on a single node or small multi-node setup. The goal is a credible, opinionated, fully documented stack — not a toy example and not a production SaaS. Every architectural choice should have an ADR.

This is **not** an Apple Silicon / local-only project. Workloads run on rented cloud GPUs (Vast.ai, Lambda, RunPod). Do not assume the constraints from the parent `.claude/CLAUDE.md` around `ollama`, Apple Silicon, or 32 GB RAM limits apply here.

## Planned stack

| Layer | Component |
|---|---|
| Inference engine | vLLM |
| Gateway / auth | TBD (see ADRs) |
| Observability | Prometheus + Grafana + OpenTelemetry |
| Load testing | TBD (`loadtest/`) |
| Eval harness | TBD (`evals/`) |
| Deploy | Docker Compose + Kubernetes manifests |

## Repository structure

Directories are added as work progresses. The intended layout:

```
deploy/          # Kubernetes / Docker Compose manifests
serving/         # vLLM config and wrappers
observability/   # Prometheus, Grafana, OpenTelemetry config
loadtest/        # Load testing harness and scenarios
evals/           # Quality evaluation under load
scripts/         # Reproducible setup and teardown
docs/
  architecture.md   # High-level architecture (evolves with code)
  adr/              # Architecture Decision Records
  writeups/         # Long-form learning posts
```

## Build and run commands

> No runnable stack exists yet. The target entrypoint will be `make up`. Update this section as components land.

## ADR conventions

All significant decisions live in `docs/adr/`. The template is at `docs/adr/README.md`. Key rules:

- File naming: `adr_NNN_short_noun_phrase.md`
- Status field must be one of: `Draft | Proposed | Accepted | Deprecated | Superseded`
- The **Alternatives Considered** table is mandatory — "why rejected" must be specific, not vague ("too complex" is not acceptable)
- The **Consequences** section must name real downsides; an ADR with no negatives is a marketing document
- When an ADR is superseded, update both the old and new files

## Design principles (held to strictly)

1. Honest tradeoffs over claimed perfection — every decision document calls out what the choice gives up.
2. Reproducibility is part of the artifact — if it can't be run, it doesn't exist.
3. Negative and surprising results are first-class — a disproven assumption is more valuable than a confirmed one.
4. Scope discipline — small things done well over large things done badly.
5. Writing is half the work — code without explanation is half-shipped.

## What to avoid

- Do not present local load-test numbers as production capacity evidence.
- Do not add components without a corresponding ADR or at least a clear rationale in the PR.
- Do not assume CUDA availability is guaranteed — document GPU/driver requirements explicitly.
- Do not create `docs/writeups/` drafts until the underlying code or experiment actually exists.