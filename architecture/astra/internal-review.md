# Internal review before independent freeze

Completed 2026-10-07. Review was performed only within the Astra team, on its own working draft, before any cross-review. No recommendations or other architect work was consulted.

The principal integrated these corrections:

1. Identity continuity and account changes are conditional on observable evidence; gaps can suspend strong capabilities.
2. Observation timestamps identify clock/boot domains, receipt time and restart freshness rules.
3. Resource-set acquisition, cancellation, bounded waits and non-preemptible gesture portions prevent indefinite lease ownership.
4. Browser projection declares visual-context policy for blending, effects and backdrop dependencies.
5. Frame-associated target incarnation, not geometry alone, protects against same-position replacement.
6. Browser security UI and activation-sensitive operations use an explicit authority channel.
7. Managed applications retain original nested browsing contexts and frame relationships.
8. Projection paint eligibility and hit testing agree when original occluders are omitted.
9. Page visibility/focus semantics are distinct from compositor frame-demand scheduling.
10. CEF source-resource use must finish within its permitted callback lifetime unless an explicit contract extends it.
11. Case macrosteps use a canonical local commit order and snapshot-consistent graph frontier; partial computation is not committed on budget exhaustion.
12. Late asynchronous results retain request/attempt/program/dependency identity and cannot advance superseded state.
13. Grants, approvals, bindings and preconditions are revalidated at dispatch; revocation stops queued effects.
14. In-flight effects retain their issuing version through migration and remain subject to reconciliation.
15. The illustrative verification gate requires complete coverage and policy satisfaction, with approvals bound to the current review attempt and dependencies.
16. Publication explicitly awaits a staged-artifact receipt and releases that exact revision; recovery preserves completed steps.

These are architectural specification improvements. They are not empirical validation of an implementation. No conformance experiments, app instrumentation or performance benchmarks were run.

Final document checks: approved intent/brief/seven findings still match their manifest entries; local supporting links resolve; code fences balance; no TODO/TBD placeholders remain. Primary-source documentation supports factual mechanism claims; proposed extensions and acceptance criteria are labeled as proposals.
