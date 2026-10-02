# eigentopic

An evolving topic map built from accumulating material, with evidence and change history preserved.

Compare new material with existing topics and, when needed, revisit earlier evidence to revise topic names and boundaries. Alongside the current map, keep a record of **what changed and which evidence drove the change**.

The name `eigen` evokes distinctive directions that emerge from the material. It does not imply eigenvector computation or the adoption of a particular numerical algorithm.

## Current status

**This is a design draft. There is no runnable collector, topic inference engine, or UI yet.**

- [DESIGN.md](DESIGN.md): scope, record model, update sequence, validation criteria, and open decisions
- [examples/synthetic-batch.json](examples/synthetic-batch.json): entirely fictional example records
- [.gitignore](.gitignore): rules that exclude real inputs, generated artifacts, personal settings, and secrets from tracking

## Initial scope

Start by designing the **slow layer**: receive material from an existing external collector and gradually update the topic map.

1. Normalize inputs and identify duplicates.
2. Create observations linked to their sources.
3. Group similar observations and compare them with existing topics.
4. Revisit the interpretation of relevant earlier material and topics in light of new evidence.
5. Preserve topic versions, previous maps, meaningful changes, and the evidence behind them.

Let topics emerge from the material rather than fitting them only into a predefined taxonomy. The specific decision methods and quality criteria for automatic topic creation, renaming, splitting, and merging remain undecided.

## Personal views

Different users may want different ordering and explanations for the same evidence. Initially, personalization applies to **display weighting and explanation style**. Avoid designs in which user preferences alter source material, provenance, or contrary evidence.

Hiding a topic must be reversible, and its records must remain intact. A policy that suppresses the future creation of similar topics is a separate decision. Whether users share a corpus or keep separate corpora, and how sharing would work, remain undecided.

## Public repository and real data

Keep only the design and reviewed synthetic examples in this repository. Store real source material, observations, topic maps, snapshots, change records, user profiles, execution logs, and generated artifacts in ignored local paths.

The recommended runtime location is under `runtime/`. `.gitignore` only excludes untracked files from Git; **it does not provide backups, encryption, or access control, and it does not remove content that has already been committed.** Review `git status` and staged changes before publishing. Keep real data out of the commit history.

The location of separate private backups and the retention and recovery policies will be decided during implementation. A license has not yet been selected.

## Design starting points

- Blei & Lafferty (2006), [Dynamic Topic Models](https://www.cs.columbia.edu/~blei/papers/BleiLafferty2006a.pdf): changes in topic representations over time
- Pirolli & Card (2005), [sensemaking research](https://www.researchgate.net/profile/Peter-Pirolli/publication/215439203_The_sensemaking_process_and_leverage_points_for_analyst_technology_as_identified_through_cognitive_task_analysis/links/02bfe50f09ca94efc0000000/The-sensemaking-process-and-leverage-points-for-analyst-technology-as-identified-through-cognitive-task-analysis.pdf): revising representations by moving between material and interpretation
- [epistemic-protocols](https://github.com/jongwony/epistemic-protocols/tree/10abcb37573c187b3fc23d5dc834519475b920b2): preserving source material, reinterpreting it, abstracting from examples, and making open decisions explicit
- [explain](https://github.com/jongwony/cc-plugin/blob/15584bc0cf1f4e50e040ae167d5edb1c0397ccce/explain/skills/explain/SKILL.md): explaining components and reasons for change at a level suited to the reader's understanding

[DESIGN.md](DESIGN.md#references-and-scope-of-application) distinguishes what each source supports from the additional design choices made in applying it to eigentopic.
