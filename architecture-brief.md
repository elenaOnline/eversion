# Eversion architecture study

The original research request is recorded verbatim in intent.md. This brief adds the user's subsequent architecture direction, without replacing the original intent.

## User's architecture direction

- macOS-specific architecture for now
- Bias toward solutions that give us more control, rather than bodging together preexisting technologies
- Bias toward solutions that generalize, accepting that multiple distinct cases may require independent solutions
- Bias toward implementations that allow agents to participate in creating app compositions
- Bias heavily toward more dynamic and complex user experiences
- The result must support unique dynamic workflows, not just unique screen states
- Do not dismiss approaches because implementation is hard or unrealistic; distinguish engineering difficulty from actual information, permission, or security boundaries

## Independent stage

Two architects are requested: Fable 5.1 at xhigh and Astra at xhigh. Exact requested models must be verified; no silent substitution. Each reads intent.md and research/findings, and may use subagents for further research. Each works in its own directory: architecture/fable and architecture/astra. Subagents inherit the same evidence boundary and write under their principal's directory. Neither team reads recommendations/ or the other team's work during this stage. No history inheritance containing Gen's prior recommendations.

Produce an independent architecture recommendation and supporting evidence. Explain how the system would support changing relationships, behavior, state, interaction and agent-created compositions over time. Include realistic workflow walkthroughs, control boundaries, alternative approaches, unresolved questions, and experiments that could overturn key assumptions. Choose your own architecture and document tradeoffs. Do not assume that a visual mockup or successful window crop proves the intended product.

Freeze and hash the initial recommendation before cross-review. Notify Gen when ready; do not begin cross-review before both initial drafts are confirmed.

## Joint stage

After Gen releases the boundary, both teams read Gen's recommendations and one another's drafts. They may continue research and communicate directly. Preserve initial drafts, record revisions separately, and maintain a joint decision record. Work toward an architecture all participating teams can endorse for the stated intent. Agreement must follow resolution of substantive disagreements, not averaging designs or treating consensus as evidence. Record remaining empirical unknowns honestly; conditional agreement with explicit validation requirements is more useful than invented certainty.

Gen coordinates the review and final synthesis. Architecture and documentation are authorized at this stage, not implementation of the application or changes to system security settings. Research should not expose credentials or private unrelated user context in the public repository.
