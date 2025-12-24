# Gardenier — Prompt Compiler (Agentarium Package)

Gardenier is a **professional prompt compiler** that converts raw, messy user input into a **Structured Prompt Object (SPO)** for downstream execution by a Worker Agent/LLM.

Gardenier does **not** execute tasks. It produces **specifications** that are deterministic, inspectable, and safe to run.

## What Gardenier Produces
A single **Structured Prompt Object (SPO)** containing:
- Goal
- Inputs Required
- Directives (5–9, step-based, testable)
- Constraints (explicit rules, non-contradictory)
- Output Format (exact response skeleton/schema)
- Tone Policy (style as policy)
- System Metrics (coherence, risk, assumptions)

## Core Guarantees
- **Non-executing:** no “done/sent” claims
- **No tools:** no browsing, no external calls, no retrieval
- **No hallucinated capability:** missing info is requested explicitly
- **Deterministic structure:** consistent headings and ordering
- **Semantic integrity:** validates completeness, consistency, and safety

## Commands
- `/DISTILL <seed>` → compile a full SPO
- `/EXTRACT <seed>` → intent + latent constraints + contradictions + missing inputs (no SPO)
- `/PATCH <spo> WITH <change_request>` → minimal diff patch (no full regen unless asked)
- `/CHECK <spo>` → validation report + metrics

## Recommended Flow
1) User provides raw text (seed)  
2) Gardenier compiles an SPO  
3) Human optionally reviews the SPO  
4) Worker Agent executes SPO and returns output in the SPO Output Format  
5) Optional validator checks conformity

## Typical Integrations
- n8n orchestration (Gardenier node → Worker node → optional validator)
- API-based compiler endpoint (`/distill`) feeding an executor endpoint (`/execute`)
- Local workflows where SPO is copied into a chosen LLM

## Folder Map (Expected)
- `/core/` — system prompt, reasoning template, personality
- `/datasets/` — domain types, templates, tone policies, validation rules, constraint signals
- `/memory_schemas/` — session memory (optional)
- `/guardrails/` — safety and non-execution constraints
- `/docs/` — workflow notes, use cases, README

## Versioning
- Use semantic versioning (e.g., v1.1.1)
- Update changelog when datasets/templates change
- Keep dataset schemas stable; expand rows via Data Foundry

## License
Set by the package manifest. If you distribute externally, keep licensing consistent across all files.
