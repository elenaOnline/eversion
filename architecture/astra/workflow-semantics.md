# Workflow semantics for Eversion

Recommend an Eversion-owned workflow runtime whose fundamental object is a **versioned composition of entities, relationships, behaviors, and effects**. A canvas is one view of that composition. The durable runtime must remain useful when every originating window is closed, while acknowledging that particular capabilities may then become unavailable.

The following is an independent design proposal, not an assertion of demonstrated compatibility. I read the permitted intent, brief, and seven findings; I did not read recommendations, another architect’s work, or unrelated project context. No implementation, system changes, or Git writes were made.

## 1. Own a small semantic kernel, not a universal application model

Use four cooperating parts:

| Part | Owns | Must not pretend to own |
|---|---|---|
| Evidence graph | Observations, identities, relationships, provenance | Unobserved source state |
| Derivation engine | Pure transformations and materialized views | External side effects |
| Case runtime | Per-entity lifecycle, decisions, waits, deadlines | Source application transactions |
| Effect broker | Authorized commands, receipts, reconciliation | Universal rollback or exactly-once remote execution |

The intermediate representation should describe all four. It must also declare presentation bindings, but presentation cannot define execution semantics.

A composition can introduce its own domain: `Story`, `Claim`, `Excerpt`, `Review`, `Revision`, `Publication`. These objects need not exist in any source application. A `Claim` may connect a paragraph in an editor, a PDF region, an issue in a web app, and a reviewer’s decision. Eversion owns the claim and its relationships; the editor owns the paragraph.

This is substantially different from making every application conform to a universal document schema. Preserve source-specific types and translate only where a workflow needs correspondence.

## 2. Identity is an evolving claim, not a selector

Distinguish three identities:

- **Intent identity:** the enduring Eversion object, such as “the central argument of this article.”
- **Source identity:** an object within an account, application, document, or project.
- **Incarnation identity:** the currently live handle, DOM node, AX element, editor buffer, or runtime object.

A source reference should carry an account/document scope, native identifier when available, incarnation generation, revision, and a locator bundle. Locators provide evidence for rediscovery; they are not identity by themselves.

`correspondsTo` must be an explicit relationship with provenance, binding revision, and status: confirmed, candidate, ambiguous, invalidated. Do not silently merge two people because their names match or redirect a deleted task to the next row occupying its screen position.

For text and spatial anchors, combine a source version with structural context, quoted content, neighboring content, and native anchors where available. W3C’s annotation model distinguishes quote selectors, position selectors, and representation state, explicitly describing position selectors as brittle under edits. That supports retaining multiple forms of anchoring evidence rather than inventing a universal permanent selector. [W3C Web Annotation Data Model](https://www.w3.org/TR/annotation-model/#selectors)

A particularly important rule: **an issued effect binds to an immutable target snapshot**. Rediscovering a new incarnation can repair future commands; it must not retarget an already dispatched command to a newly guessed entity.

Deletion also needs evidence. A missing virtualized row means “not observed,” not “deleted.” An adapter can establish deletion through a tombstone, authoritative lookup, or a complete enumeration over a declared scope. Eversion should require such a completeness witness before interpreting absence as fact.

## 3. Separate observed, intended, and accepted state

An observable field should resemble:

```text
Observation<T> {
  subject, field, value
  sourceRevision?, incarnation
  observedAt, validFor?
  evidenceKind, evidenceReference
  coverage, provenance
}
```

Maintain these distinct stores:

- **Observed state:** what a particular channel reported.
- **Intended state:** what a person or rule requested.
- **Accepted state:** what the composition has decided to rely upon.
- **Effect state:** what was attempted and what its outcome evidence establishes.

Do not reduce all uncertainty to one numeric confidence score. A value from a native API, a value inferred from pixels, and a value acknowledged by a remote service have different failure modes. Confidence cannot substitute for a missing capability.

Conditions need at least `true`, `false`, `unknown`, and `conflicting` outcomes. In particular, “all reviews approved” must not become true because a disconnected adapter currently returns an empty list. Missing membership coverage prevents completion.

Source authority should be assigned per field and operation. Eversion can own an editorial decision while a native application owns document contents and a service owns delivery status. Bidirectional mappings require declared round-trip behavior. Lens research formalizes useful laws for view updates, but those laws must be established for the chosen transformation; a flattened representation does not acquire an unambiguous inverse simply because an agent proposes one. [Foster et al., *Combinators for Bi-Directional Tree Transformations*](https://www.cis.upenn.edu/~bcpierce/papers/lenses-toplas-final.pdf)

## 4. Combine dataflow, statecharts, and durable effects

Pure incremental dataflow is the right substrate for changing views: “claims supported by this source,” “reviews invalidated by this edit,” or “work currently blocked by a missing capability.”

It is a poor sole model for behavior. A formula that becomes true repeatedly must not repeatedly send a message. A state machine alone is also insufficient: encoding every combination of document, participant, and source status creates an unmanageable state space.

Use hierarchical statecharts **per workflow case**, over a shared evidence graph. An article can have independent research, drafting, review, and publication regions. Each claim may have its own smaller case. W3C SCXML provides precise precedent for hierarchical and parallel states and run-to-completion processing; parallel regions need not mean concurrent threads. Eversion should own a compact interpreter with similarly specified semantics rather than inherit an entire integration product. [W3C SCXML](https://www.w3.org/TR/scxml/)

Each committed event produces one deterministic local macrostep:

1. Append the observation or decision.
2. Update pure derivations.
3. Resolve enabled transitions.
4. Persist state changes and effect intents atomically in Eversion’s store.
5. Release effect intents to the broker.
6. Receive outcomes as new events.

Replay reconstructs local decisions; it does not repeat external actions. Durable histories are a demonstrated recovery mechanism in workflow systems, but their existence does not establish atomicity with an arbitrary desktop application. [Temporal event-history documentation](https://docs.temporal.io/encyclopedia/event-history)

Timers, agent responses, randomness, and external reads enter as recorded events. Long-running work is suspended explicitly across sleep, logout, missing applications, and revoked permissions.

Keep continuously controlled parameters separate. A reversible slider or transport follower can use an explicit controller with sampling rate, deadband, saturation, feedback expectations, and loss-of-observation behavior. It should not share the retry semantics of sending an email.

## 5. Make weak effects first-class

A command contract should state:

```text
Command {
  operation, targetBinding, expectedRevision?
  arguments, authorizationScope
  idempotencySupport, conflictPolicy
  successEvidence, reconciliationProcedure
  compensation?, deadline
}
```

Track at least:

```text
prepared → dispatched → confirmed
                      ↘ rejected
                      ↘ outcomeUnknown → reconciled
```

“Dispatched” is a local fact. “Confirmed” means the declared success evidence was received. A successful accessibility call may establish that an invocation was accepted; it does not necessarily establish the application’s intended result.

The broker must choose behavior by capability:

- If the source supports a stable idempotency key, retry with the same key.
- If it supports authoritative outcome lookup, reconcile before retrying.
- If it supports revision-conditional writes, apply that condition.
- If none exists, preserve an unknown outcome and request resolution rather than duplicate a potentially irreversible action.

Preflight observation does not eliminate the read-check-write race. An Eversion lease can serialize Eversion’s own commands; it cannot exclude the user or another app from changing the target. Where the source cannot perform a conditional write, the contract must explicitly admit that limitation.

Cancellation prevents effects not yet dispatched. Compensation is a new effect, which can itself fail. “Undo workflow” should therefore offer concrete operations with established reversibility, not a fictional global rewind.

Coordination should concentrate around consequential decisions, not every observation. The CALM result supplies a useful distinction: monotonic deductions can have coordination-free implementations, while conclusions involving absence or replacement need stronger treatment. This is a design guide for the local runtime, not permission to infer completeness from partial app observations. [Hellerstein and Alvaro, *Keeping CALM*](https://arxiv.org/abs/1901.01930)

## 6. Dynamic membership and live editing need explicit semantics

Long workflows change their own topology. Two details are easy to miss:

**Cohorts:** “Review every claim” could mean claims present at the start, all claims discovered before a cutoff, or a continuously changing membership. Encode the choice. Otherwise newly added work silently escapes a completion gate, or an expanding set can prevent completion forever.

**Migrations:** Pin every active case to a behavior version. A changed rule needs an explicit migration: pending cases only, future cases only, or a tested transformation of existing states. Preserve old decisions and receipts. Approvals should refer to the content and dependencies actually approved; consequential edits invalidate those approvals selectively.

Eversion should expose this as an ordinary interaction: “Apply this rule to new work; also move these three unfinished claims back to verification.” The user edits behavior without having to understand replay internals.

## 7. Agents should be compiler assistants and adapter researchers

An agent’s output should be a reviewable change to the composition program, with examples and tests, rather than an opaque sequence of tool calls.

A proposed workflow passes through:

1. Intent and example collection.
2. Capability discovery against the actual installed apps.
3. Typed IR generation.
4. Static checks: authority, effect ownership, unknown handling, feedback cycles, units, cohort semantics.
5. Trace simulation, including source changes and failures.
6. Deployment of a versioned composition.

Agents can also propose adapters. Each adapter package should specify supported runtime/version fingerprints, identity rules, schemas, observation coverage, operations, and evidence required for success. Recorded examples become regression fixtures. Deliberate counterexamples establish where the adapter must refuse an action.

Do not promote a generated adapter merely because it completed one demonstration. Test relaunch, duplicated labels, stale handles, app updates, interrupted writes, and concurrent human activity.

A repairing agent can diagnose a failed locator and propose a new binding. It must not silently widen authorization or reinterpret a pending effect. Keep agents off deterministic scheduling and real-time paths; record their suggestions as external events.

## 8. An evolving workflow: research becomes a publishable essay

A writer begins with browser sources, PDFs, and an unsaved editor document. She asks Eversion to build an argument workspace.

The agent creates `Claim`, `Excerpt`, `Source`, and `Review` types. Selecting text in the editor creates a claim connected to its current buffer revision. This must use a live-buffer adapter where available: VS Code explicitly distinguishes document versions, closure, and unpersisted changes; a filesystem watcher cannot stand in for that state. [VS Code extension API](https://code.visualstudio.com/api/references/vscode-api#TextDocument)

Selecting a source passage adds an excerpt with anchoring evidence. The workspace immediately presents supporting and contradictory evidence beside the claim. Browser portals remain available for original interactions.

Later, the writer splits one claim into two. Eversion retains the original’s history and records derivation relationships. It proposes which excerpts still support which new claims. Existing reviewer approval does not automatically transfer.

A source page changes. The relevant excerpt becomes stale; only dependent claims enter “verification needed.” Unaffected writing continues. If the browser is offline, the old evidence remains visible with its recorded representation, while publication guards requiring fresh verification remain unknown.

A colleague’s feedback introduces a new requirement: every quantitative claim needs two independent sources. The agent proposes a behavior patch and simulates it over the current essay. Three pending claims gain a verification step; one already published claim becomes a follow-up case. The published artifact is not silently rewritten.

The writer then forks the essay into a short version and a technical version. They share sources and some claims but have independent argument structure, approval scopes, and publication cases. Selecting a claim changes several views; it does not globally overwrite the other branch’s selection.

Publication is a durable case: freeze a content revision, generate outputs, verify them, stage a destination draft, then perform the authorized release. If the destination commits but its response is lost, the case enters “outcome unknown.” Restarting the Mac resumes reconciliation. It does not publish twice.

This is one continuously evolving program assembled around the writer’s tools, not a succession of bespoke screen layouts.

## 9. Tradeoffs and decisive experiments

The proposed kernel increases semantic work compared with a graph of scripts. Its value depends on substantially lowering repair and explanation costs over weeks.

Test these assumptions directly:

- **Identity:** Rename, duplicate, reorder, delete, reopen, and virtualize 100 bound objects. Any silent wrong-target write fails the identity architecture. Ambiguous bindings should suspend locally.
- **Unknown outcomes:** Crash at every boundary around an irreversible command. Pass only if recovery never reports unsupported success and never blindly duplicates an unknown effect.
- **Live authority:** Change an unsaved buffer while disk, preview, and remote copy disagree. Pass if every derived result identifies its actual input revision and approval validity follows that revision.
- **Workflow evolution:** Introduce new members, rules, and branches during active cases. Pass if the resulting obligations match explicit cohort and migration choices.
- **Adapter transfer:** Ask an agent to adapt a workflow to three materially different applications. Measure how much IR survives and how much domain mapping needs human input. Extensive per-app behavioral rewrites would falsify the hoped-for generalization boundary.
- **Utility:** Run the essay scenario for two weeks, including interruptions and corrections. Measure time spent understanding and repairing Eversion versus completing the work. A polished canvas with higher repair cost than the original workflow fails the product thesis.

The strongest uncertainty is whether sufficient identity and outcome evidence can be obtained economically across valuable applications. The architecture should make that question measurable per capability, while still providing useful compositions when only part of a workflow can be automated.
