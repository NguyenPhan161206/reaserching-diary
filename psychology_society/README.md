# Psychology & Society — Why People and Systems Behave As They Do

## Purpose

This folder holds the behavioural and social-science explanations that sit **underneath** technical decisions. Every note here answers one question: *why does a person or a group do that?* — because that question is what turns a technical fact into an engineering judgement.

The folder exists because three different technical projects kept hitting the same wall: a system behaves correctly and the humans using it still do the wrong thing. Recommenders that cannot be trusted, hiring filters that encode a trait into a threshold, product features that infer a psychological state. Each of those is a psychology problem wearing a machine-learning costume.

**Language:** all notes are written in English per AGENTS.md Section A6. Vietnamese appears only inside marked quotations of a source term being taught (see `vocabulary/`).

**Template:** Template C — Social Science & Psychology. Every note must satisfy the Section H checklist: cited studies with sample sizes and methodology, a real-world case, **counter-arguments**, ethical implications, and a measurable outcome somewhere.

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | [The Dunning–Kruger Effect](01-dunning-kruger-effect.md) | C (Social) | Why do incompetent people systematically overestimate themselves — and why is the explanation still contested? |
| 02 | [The Hedonic Treadmill](02-hedonic-treadmill.md) | C (Social) | Why do material gains stop improving wellbeing within about a year, while a bad event still hurts for years? |
| 03 | [The Tragedy of the Commons](03-tragedy-of-the-commons.md) | C (Social) | Why do individually rational actors destroy a shared resource, and what is actually different between the cases that collapse and the ones that don't? |
| 04 | [The Big Five Personality Model](04-big-five-personality-model.md) | C (Social) | How did psychology turn a list of adjectives into five measurable dimensions — and what can those dimensions not tell you? |

### Progression

The first three notes are **phenomena**: a cognitive bias, a wellbeing pattern, a collective-action failure. Each is a single mechanism, studied in depth. **Note 04 is a framework** — the measurement instrument that lets you talk about individual differences at all, and the note that connects the folder to `ai_ml/` and `vocabulary/`.

A fifth note on **trait vs. state** is the obvious next step: it is the statistical spine underneath note 04 and it has no home yet (see Backlog).

---

## 🔗 Cross-References

- [[vocabulary/13-personality-traits-neutral-positive-negative.md]] — the whiteboard vocabulary note that note 04 explains; also `sensitive` / `lazy` / `greedy` as the *everyday* versions of measurement problems
- [[ai_ml/07-affective-computing-and-emotion-recognition.md]] — trait inference (psychometrics) vs. state inference (machine learning): different validity profiles, different consent requirements, same person
- [[ai_ml/01-ai-in-business.md]] — where a behavioural insight turns into a product decision
- [[economics_business/04-network-effects.md]] — the group-level counterpart to the individual-level models here; it explains individual behaviour *without* any trait appeal
- `psychology_society/` notes are cited by `vocabulary/` notes more often than the reverse: the vocabulary folder is where the *language* of behaviour is pinned down, and this folder is where the *mechanism* behind it lives

---

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| **High** | **The trait–state distinction and how it is measured** | Note 04 asserts within-person variance is substantial; the measurement literature behind that deserves its own note, and it is the statistical spine of everything else here |
| **High** | **Social proof, authority, and commitment — the three mechanisms of influence** | Cialdini's set is the most-used and least-verified claim in applied psychology; separating the robust parts from the oversold parts is real work, and it maps directly onto growth engineering and AI product design |
| Medium | Loss aversion and prospect theory | Kahneman & Tversky's core result, and the one most misapplied in business writing; Tversky & Kahneman's 1992 *cumulative prospect theory* fixes the original formulation |
| Medium | Identity foreclosure, moral foundations, and political behaviour | Explains *why* the "personality inference → micro-targeting" pipeline works, which `ai_ml/07` asserts but cannot support from this folder yet |
| Medium | Cognitive load and interface design | The bridge note between this folder and `dev_foundation/web_basics/`; why interfaces fail people who are already under load |
| Low | Bystander effect and diffusion of responsibility | Over-taught, heavily replicated, and its canonical explanation has been substantially revised — a good "what changed since the textbook" case |

---

*Last updated: 2026-09-30*
