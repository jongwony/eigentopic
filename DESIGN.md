# eigentopic design draft

Status: **proposed**. The behavior, data structures, and tests in this document do not describe implemented features. The technology stack, storage format, similarity calculations, and models remain undecided.

## Purpose

Build a map whose topic names and boundaries change as material accumulates. It should be possible to trace which material was read, which observations were made, and what changed from the previous understanding.

Rather than fixing a complete taxonomy at the outset, find recurring relationships in the material and give them provisional names. When a new example does not fit an existing name, reconsider both the grouping and the name. Preserve source material and version its interpretations.

## Scope

### First stage: the slow layer

This draft covers receiving batches from an external collector and updating an accumulating topic map. The collection service itself, real-time notifications, service-account connections, deployment, and changes to existing automations are out of scope.

The first deliverable has three goals:

1. Current topics and the supporting, opposing, and unresolved evidence for each
2. A change record connecting the previous map to the current one
3. Explanations that can be traced back to their sources

Before implementation, check this contract with a small synthetic dataset. Avoid a design that requires real user data in order to validate it.

## Records to store separately

The names below form a conceptual model. The fields and serialization format are not a finalized API.

| Record | Role and minimum links |
| --- | --- |
| `SourceRevision` | A particular version of input material. Contains a stable source ID, collection time, original-document identifier, source version or hash, and references to the original wherever available. Distinguishes revisions at the same URL. |
| `Observation` | A statement or relationship read from source material. Links to the source revision and location, and distinguishes direct observations or quotations from model inferences. Corrections do not overwrite earlier records. |
| `TopicVersion` | One version of a topic ID. Contains a name, scope description, linked observations, supporting/opposing/unresolved relationships, and a reference to the previous version. Keeps names separate from IDs. |
| `Change` | An event such as creation, renaming, scope revision, splitting, merging, or correction. Links before-and-after versions, triggering observations, the reason for the change, and proposed/applied status. |
| `Snapshot` | The map finalized by one update. Records the input batch, the exact set of topic versions, the previous snapshot, and a run identifier. |
| `ViewProfile` | A user's display weights, explanation detail level, hidden topics, and similar preferences. This view state is independent of evidence records. |
| `RunRecord` | Records inputs, processing configuration, processor/model versions, and whether the update succeeded. Provides a basis for deciding how to reprocess the same inputs and recover from failures. Does not record secrets. |

Represent revised interpretations as new records. Requests to correct or delete source material, and retention periods, need separate policies; do not promise indefinite retention unconditionally. When source material can no longer be retained, consider showing the reason and the resulting limits on verifying its interpretations, to the extent permitted.

## Batch update sequence

The following is a processing contract to validate during implementation.

1. **Validate inputs and identify duplicates**: use the external collector's stable IDs, source revisions, normalized content, and related signals. Do not inflate the count of independent evidence with recollected material, repeated citations, or copied documents. Do not discard distinct material based on semantic similarity alone.
2. **Extract observations**: attach a source revision and location to each observation. Separate what the source says from relationships inferred by the system. Leave matters unresolved when information is insufficient.
3. **Compare with existing topics**: retrieve candidate observations and topics that may be relevant. If no suitable grouping exists, propose a new topic. Similarity scores help find related material; they do not replace factual judgment.
4. **Revisit earlier evidence**: reread earlier observations to determine whether new examples call for revised names, boundaries, or relationships. Preserve material that conflicts with the current interpretation, alternative explanations, and what remains unexplained.
5. **Create new versions and change records**: link each change to the material that prompted it. For splits and merges, explicitly record relationships between prior and successor topic IDs so that old records remain discoverable.
6. **Finalize the snapshot**: choose a way to finalize an update's records together, so an intermediate failure cannot leave only part of the map on new versions. The last finalized snapshot must remain readable until the new one is finalized.
7. **Explain meaningful changes**: explain new topics, revised interpretations, new opposing evidence, splits, and merges rather than merely reporting new-document counts. If nothing changed, say so. Keep source material distinct from revised interpretations.

This does not mean rereading the entire history for every batch. Which earlier evidence to revisit, and how to measure omission risk and processing cost, remain undecided. Whether to apply provisional candidates automatically or require human review must also be decided before implementation.

## Change illustrated with a synthetic example

The [example JSON](examples/synthetic-batch.json) contains hand-authored fictional records, not actual collection results. All URLs use the example-only `.invalid` domain.

- The first batch contains notices about an unattended book-return box and nighttime returns at a fictional library. The provisional topic is `After-hours book returns`.
- The next batch adds a notice about collecting reserved books outside opening hours. Revisiting the first two pieces of evidence leads to a proposal to broaden the topic to `After-hours book access`.
- The previous name and version remain available. The new explanation links all three observations and leaves open whether returns and pickups would be more useful as separate topics.
- This grouping interprets the services as having similar purposes. It does not establish that all three services use the same equipment or have the same operator.

The same structure can be tested with software-related material. Log-collection examples and incident-diagnosis examples might be grouped together, but similar terminology alone cannot establish a shared cause or solution. Distinguish a grouping's usefulness from the strength of evidence for a claim.

## Boundaries of personal views

Initially, personalization applies to display order, emphasis, and the background and detail needed in explanations. A view does not change the evidence or change history of the same map snapshot.

- **Hiding**: remove a topic from the default display. This is reversible; source material and previous snapshots remain intact.
- **Suppressing future creation**: a policy that restricts the creation of similar topics in the future. This differs from hiding and needs separate designs for scope, removal of the restriction, and audit records. The current draft does not apply this policy automatically.
- **Suggesting perspectives**: offer ways of reading the material that may matter to the user without treating them as confirmed user intent. Leave important unresolved choices as questions.

Whether multiple users share a corpus or use separate corpora remains undecided. Separating `ViewProfile` does not itself establish data sharing or access permissions. Before implementing sharing, define per-source access permissions, isolation between users, and deletion policies.

## Public repository boundary

Use Git to manage the design and reviewed synthetic examples. Keep real source material, normalized data, observations, maps, change histories, snapshots, profiles, run records, and caches in ignored paths such as `runtime/`. Exclude `.env` and files containing secrets as well.

`.gitignore` is not a security boundary. Already tracked files, force-added files, and files mistakenly saved elsewhere can become public. Before publishing, review the file list and staged content, and keep real data out of the commit history. Separate backup, encryption, and recovery procedures for ignored paths remain undecided.

Rights to store source material, retention periods, handling of sensitive information, and what material may be sent to model providers must be decided before implementation. Topic names and summaries can themselves contain sensitive information, so excluding only the original source material is insufficient.

## Proposed validation criteria

These are **acceptance tests for a future implementation**, not functional tests that currently pass.

- **Idempotency**: reprocessing the same batch with the same configuration does not create duplicate observations or change events. Reinterpretation with a different model or configuration is recorded as a separate run.
- **Source traceability**: every evidence link resolves to an existing source revision and location. Broken references explicitly indicate that verification is unavailable.
- **History preservation**: earlier snapshots, including their names and scopes at the time, can be reconstructed after renaming, correction, splitting, or merging.
- **Repeated citations**: multiple documents citing the same original are not counted as multiple independent pieces of evidence. Uncertain duplication relationships remain distinct from confirmed duplicates.
- **View invariance**: changing display weights or hiding preferences does not alter source, observation, or topic-version content.
- **Restoring hidden topics**: showing a hidden topic again restores its original history and links. Creation suppression must be a separate state.
- **Update atomicity**: the last finalized snapshot remains consistently readable after an intermediate failure.
- **Effective reconsideration**: when synthetic input introduces a new example that breaks the current scope, the result explains why the scope was or was not revised, with references to earlier evidence.

At the design stage, check the example JSON's syntax and reference links, representative `.gitignore` paths, and the list of files to publish. A consistent synthetic example does not demonstrate actual inference quality.

## Decisions before implementation

1. Corpus boundaries: a single user, shared material, or separate material for each user
2. Input contract: external collector IDs, revision and deletion notices, and batch boundaries
3. Processing approach: representation models, retrieval of related material, and criteria for topic candidates and renaming
4. Reconsideration scope: selecting relevant earlier evidence, recomputation cost, and omission evaluation
5. Authority to apply changes: automatic application, user review, and conditions for deferral
6. Retention and recovery: rights to store source material, retention periods, private backups, and deletion policies
7. Implementation and deployment approach, and license

## References and scope of application

### Academic sources

**Blei, D. M. & Lafferty, J. D. (2006). _Dynamic Topic Models_. ICML.** [Paper](https://www.cs.columbia.edu/~blei/papers/BleiLafferty2006a.pdf)

Presents a probabilistic model in which topic word distributions evolve across time slices. For eigentopic, it provides a starting point for updating topic representations over time. The paper's model uses a fixed number of K topics. We do not claim that it provides automatic topic birth, splitting, or merging, and we have not adopted its algorithm.

**Pirolli, P. & Card, S. (2005). _The Sensemaking Process and Leverage Points for Analyst Technology as Identified Through Cognitive Task Analysis_.** [Paper](https://www.researchgate.net/profile/Peter-Pirolli/publication/215439203_The_sensemaking_process_and_leverage_points_for_analyst_technology_as_identified_through_cognitive_task_analysis/links/02bfe50f09ca94efc0000000/The-sensemaking-process-and-leverage-points-for-analyst-technology-as-identified-through-cognitive-task-analysis.pdf)

Describes iterative movement between gathering material and refining representations and hypotheses. It informs eigentopic's step of revisiting earlier evidence. As a conceptual model based on an initial cognitive task analysis of a limited set of analytical tasks, it is not evidence that this product or its automation has been validated.

### Public design references

The following principles summarize and adapt the reference material. The original skills and format specifications are neither copied into nor installed in this repository. Validation results from those projects are not assumed to carry over to eigentopic.

- **[Hermeneutic Cycle](https://github.com/jongwony/epistemic-protocols/blob/10abcb37573c187b3fc23d5dc834519475b920b2/.claude/principles/hermeneutic-cycle.md)**: move between interpretations of the parts and the whole while preserving source material. This draft makes that concrete through source-revision preservation, reconsideration of earlier evidence, and versioned interpretations. These specifics are eigentopic design proposals.
- **[/induce](https://github.com/jongwony/epistemic-protocols/blob/10abcb37573c187b3fc23d5dc834519475b920b2/periagoge/skills/induce/SKILL.md)**: refine structures shared by concrete examples into candidate names and scopes while preserving alternative interpretations. This informs the choice not to fix topic names in advance. The interactive protocol is not treated as an automatic clustering algorithm.
- **[/elicit](https://github.com/jongwony/epistemic-protocols/blob/10abcb37573c187b3fc23d5dc834519475b920b2/euporia/skills/elicit/SKILL.md)**: make necessary decisions and their rationale explicit while distinguishing proposals from user-confirmed intent. This helps keep open decisions, such as personal display intentions and corpus boundaries, from being settled by inferred preferences.
- **[explain](https://github.com/jongwony/cc-plugin/blob/15584bc0cf1f4e50e040ae167d5edb1c0397ccce/explain/skills/explain/SKILL.md)**: explain component roles and interactions alongside their evidence, at a level suited to what the reader already knows and wants to understand. For eigentopic, this informs explanations that distinguish source statements, interpretations, and differences from previous versions. Personalized explanations do not change the evidence.
