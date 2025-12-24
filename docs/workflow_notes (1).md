[AGENTARIUM_ASSET]
Name: Workflow Notes — Implementation Guide
Version: v1.1.1
Status: Draft

Purpose
This document explains how to implement the Gardenier package in a real environment:
- core files (system prompt, reasoning template, personality fingerprint, guardrails)
- memory
- datasets + optional knowledge map
- vector database ingestion (embed + upsert)
- execution flow using either LangChain or n8n

-------------------------------------------------------------------------------

1) Package Assembly (Core Files)

1.1 Files and roles
- core/system_prompt.md:
  The identity + operating rules of Gardenier.
- core/reasoning_template.md:
  The deterministic pipeline Gardenier follows.
- core/personality_fingerprint.md:
  The voice/behavior constraints (professional, neutral, structured).
- guardrails/guardrails.md:
  Hard safety constraints and non-execution rules.

1.2 How to combine them at runtime
When you run Gardenier, assemble a single “Gardenier Runtime Prompt” as:
A) System Prompt
B) Guardrails (hard constraints)
C) Reasoning Template (pipeline rules)
D) Personality Fingerprint (tone/behavior dial)
E) Output Format Enforcement (SPO structure)

Implementation rule:
- System + Guardrails must be treated as highest priority.
- Personality never overrides Guardrails.
- Reasoning template governs the compiler loop and output structure.

-------------------------------------------------------------------------------

2) Datasets and Knowledge Map (RAG Layer)

2.1 Required datasets (minimum)
- domain_type_catalog.csv
- latent_constraints_signals.csv
- prompt_template_catalog.csv
- tone_policy.csv
- validation_rules.csv

2.2 Optional knowledge map
A knowledge map is a lightweight entity graph describing package primitives.
Use it if you want better recall and safer expansions.
Typical entities:
- domain_type
- template_id
- tone_id
- validation_rule_id
- latent_constraint_type

Typical relations:
- domain_type -> uses_template -> template_id
- domain_type -> recommended_tone -> tone_id
- domain_type -> requires_validation -> rule_id
- latent_signal -> implies_constraint -> constraint_rule

2.3 Document normalization before embedding
Before embedding, convert each dataset row into a canonical text record.

Example record format:
[ROW]
dataset=validation_rules
rule_id=VAL_REQUIRED_SECTIONS
rule_type=completeness
severity=critical
description=...
fix_hint=...

Store metadata alongside each record:
- dataset_name
- primary_id (rule_id/template_id/tone_id)
- domain_type (if present)
- severity (if present)
- version

-------------------------------------------------------------------------------

3) Vector Database Upsert (Embed + Index)

3.1 Choose a vector DB
Any vector store works (Pinecone, Qdrant, Weaviate, Chroma, pgvector).
You need:
- an embeddings model
- a vector index/collection
- metadata filters (recommended)

3.2 Step-by-step upsert procedure
Step 1: Load CSV files
- Read each CSV row.
- Validate required columns exist (schema check).

Step 2: Convert each row to a document
- Use the canonical record format (Section 2.3).

Step 3: Generate embeddings
- For each document text, generate an embedding vector.

Step 4: Upsert into the vector DB
- Use a stable ID: {dataset_name}:{primary_id}
- Store metadata (dataset_name, domain_type, severity, version).

Step 5: Verify retrieval
- Query examples:
  - required SPO sections
  - tone policy for executive brief
  - high severity safety constraints
- Confirm top hits match expected rows.

3.3 Index strategy (recommended)
- One index/collection for the package: gardenier_knowledge
- Use metadata filtering by dataset_name to retrieve targeted signals.
- Retrieve at least:
  - templates + domain types (routing)
  - validation rules (integrity)
  - latent constraint signals (inference)
  - tone policies (style)

-------------------------------------------------------------------------------

4) Memory Implementation (Session Memory)

4.1 Memory scope rule
- Default: session-only memory (recommended).
- Store only what improves compilation accuracy.

4.2 Memory fields (minimum)
- session_id
- last_seed
- last_domain_type
- latent_constraints (carryover)
- constraints_carryover (carryover)
- tone_preference (optional)

4.3 Memory usage rule
- Memory must not override new explicit user requirements.
- Memory can only:
  - suggest default tone
  - carry constraints like no hype, strict format
  - maintain continuity across turns

-------------------------------------------------------------------------------

5) Running Gardenier with RAG (Compilation Logic)

5.1 Retrieval plan (what to fetch)
Given seed text:
A) Retrieve domain routing hints:
- domain_type_catalog + prompt_template_catalog
B) Retrieve latent constraint patterns:
- latent_constraints_signals
C) Retrieve tone options:
- tone_policy
D) Retrieve validation constraints:
- validation_rules

5.2 Compilation loop
- Parse seed -> infer candidate domain_type.
- Retrieve top-k rows per dataset (k small: 3–8).
- Compile a draft SPO using the selected template.
- Run validation checks (based on retrieved rules).
- If invalid, rewrite and re-check.
- Output exactly one SPO.

-------------------------------------------------------------------------------

6) Implementation in LangChain (Reference Build)

6.1 Components
- LLM: the model you use for Gardenier
- Retriever: vectorstore.as_retriever()
- Prompt assembly: merge core files + retrieved snippets
- Memory: session store (ConversationBuffer or custom store)
- Output parser: ensure SPO structure (regex/section checks)

6.2 Minimal steps
Step 1: Load core files as strings.
Step 2: Load vector store retriever (gardenier_knowledge).
Step 3: On each request:
  - retrieve relevant rows (filters by dataset_name)
  - assemble Gardenier Runtime Prompt (core + retrieved context)
  - call LLM
  - validate structure (required sections)
  - if fail, retry once with repair instruction
Step 4: return SPO.

6.3 Guardrail enforcement
- Hard-code a post-check that rejects outputs containing:
  - tool use claims (I searched, I emailed, I executed)
  - missing required headings
- If violated: re-run with repair to comply instruction.

-------------------------------------------------------------------------------

7) Implementation in n8n (Reference Build)

7.1 High-level workflow
Workflow: Gardenier Compiler
1) Trigger (Webhook / Chat input)
2) Load core files (static text nodes or file read)
3) Retrieve knowledge (Vector DB query node / HTTP request)
4) Assemble prompt (Set/Function node)
5) Call LLM (OpenAI/LLM node)
6) Validate output (IF node + Function validator)
7) If invalid -> Repair call (LLM node once) -> Validate again
8) Return SPO (Webhook response)

7.2 Step-by-step
Step 1: Trigger node receives {seed, optional context, session_id}.
Step 2: Retrieve from vector DB:
- Query = seed
- Filter dataset_name in batches:
  - domain_type_catalog + prompt_template_catalog
  - latent_constraints_signals
  - tone_policy
  - validation_rules
Step 3: Assemble a single prompt:
- System: system_prompt + guardrails
- Developer/message body: reasoning_template + personality + retrieved snippets
- User message: seed + context
Step 4: LLM node generates SPO.
Step 5: Validator function checks:
- required headings exist
- directives count 5–9
- no tool/action claims
Step 6: If fail -> Repair SPO to comply LLM node -> Validate again.
Step 7: Return SPO.

7.3 What to store in n8n
- session memory in a DB (Supabase/Postgres) or simple store:
  - session_id, last_domain_type, constraints_carryover, tone_preference

-------------------------------------------------------------------------------

8) How to Start Using It (Operational)

8.1 First run checklist
- Core files loaded and concatenated correctly.
- Vector DB index exists and contains embedded dataset rows.
- Retrieval returns relevant rows.
- SPO validator is active.
- Output is pasteable into Worker agent.

8.2 Example user request
Input seed:
Here’s a messy feature list for my app… make it a clean spec with milestones.

Expected output:
- domain_type = project_spec
- SPO with required sections
- directives 5–9
- output format includes a spec template

8.3 Common failure modes
- Too much retrieval noise:
  - fix by filtering by dataset_name and lowering top-k.
- SPO missing headings:
  - enforce validator + repair loop.
- Over-assumptions:
  - require more Inputs Required.

-------------------------------------------------------------------------------

9) Recommended Version Discipline
- When you expand datasets, bump dataset version and package version.
- Keep schemas stable; expand rows, not columns, unless major version bump.
- Treat templates and validation rules as core intelligence assets.
