# Software Architecture — System Design Notes

**Purpose:** Architecture patterns and system-design knowledge for building the **[Modular Company Task System](../random_ideas/06-modular-company-task-system-ai-agents.md)** idea — and for leveling up as an AI Engineer (Economics/Business) who ships real, long-lived systems.

## Coverage

| # | Note | Core Question |
|---|------|---------------|
| 01 | Event-Driven Architecture & Task Orchestration | How do tasks *link* through a solid back-end? |
| 02 | Modular Monolith vs Microservices | How do components stay *independent* without falling apart? |
| 03 | Application Architecture Models | What do monolithic, multi-tier, microservices and serverless each promise — and cost? |
| 04 | Web Performance, Caching & Bottlenecks | Where does latency hide across the layers, and what do I fix first? |

## Template

Uses **Template A: Tech Note (adapted)** — see `NOTE_TEMPLATE.md`. Every note must pass the Section H depth checklist (code/pseudocode, architecture explanation, "why it matters", career link).

## Naming Convention

- Files: `XX-descriptive-name.md` (XX = 01, 02, 03… chronological)
- Bridge notes: `bridge-topic-a-and-topic-b.md`

## Research Journey Note

These notes exist to *resolve the tensions* of the Modular Company Task System idea: independence vs integration (note 01/02), the model-choice trade-offs the course demands (note 03: the 3-tier monolith is the right default), and where latency hides in every SSR app (note 04). Treat every pattern here as a **candidate solution**, not a dogma — architecture is trade-offs made explicit.

*Last updated: 2026-10-10*