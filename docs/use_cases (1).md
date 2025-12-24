# Gardenier — Use Cases (v1.1.x)

Gardenier is most valuable anywhere you need **structured prompts** instead of conversational prompts. Below are practical use cases you can ship, demo, or integrate.

## 1) Messy Notes → Project Specification
**Input:** scattered notes, feature ideas, constraints, deadlines  
**Output:** SPO that forces a clear goal, requirements, milestones, and deliverables.

## 2) Founder Brain Dump → Product Requirements Document (PRD)
**Input:** vision + scattered requirements  
**Output:** SPO that locks requirements, assumptions, unknowns, and success criteria.

## 3) Customer Complaint → Support Response Spec
**Input:** angry/unclear customer messages  
**Output:** SPO that produces de-escalation steps, clarification questions, and resolution format.

## 4) “Make This Better” → Rewrite/Editing Specification
**Input:** vague request to improve copy/article/email  
**Output:** SPO that locks tone, length, target audience, and exact output format.

## 5) Code Snippet/Repo Issue → Debug or Refactor Spec
**Input:** logs/snippets + symptoms  
**Output:** SPO that enforces step-based debugging, safety constraints, and a clean engineering report format.

## 6) Policy/Compliance Drafting
**Input:** rules, constraints, jurisdiction notes  
**Output:** SPO that forces definitions, scope, audit requirements, and conservative assumptions.

## 7) Persona / System Prompt Compilation
**Input:** desired persona behavior, boundaries, style  
**Output:** SPO that generates a worker-ready system prompt spec with explicit constraints.

## 8) Decision Framing & Trade-off Analysis
**Input:** options, constraints, preferences  
**Output:** SPO that enforces a decision protocol, comparison table, and confidence scoring.

## 9) Research Summary Specification
**Input:** article/paper + what user cares about  
**Output:** SPO that forces summary structure, key points, limitations, and next questions.

## 10) Dataset Expansion Requests (Data Foundry Bridge)
**Input:** small CSV schema + goal  
**Output:** SPO that defines expansion rules, diversity constraints, and validation checks for synthetic data generation.

## Notes
- Gardenier is best used as a **compiler** in front of an executor model.
- Use `/CHECK` for QA and dataset-driven validation.
- For high-risk domains, prefer stricter assumptions and explicit missing-input requests.
