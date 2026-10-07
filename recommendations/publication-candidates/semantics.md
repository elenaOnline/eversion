> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Semantic interoperability for a live, composable desktop

Research date: 2026-10-07. This memo distinguishes documented capabilities from proposed architecture. No software was installed or run against a user's applications.

## Bottom line

A desktop can plausibly make existing applications feel like a single, personally assembled instrument without rebuilding every application. But the honest architecture is **live surfaces plus typed, capability-scoped bridges**, with different fidelity for different apps. Extracting an interactive panel and making its meaning available to another app are independent achievements.

There are three separable products:

1. **Live visual composition:** Present an existing app or region and route input back to the running instance. Retains behavior to the extent that capture, focus, popups, coordinate transforms and input routing work. Does not automatically reveal its domain model.
2. **Semantic composition:** Read named state, subscribe to changes, invoke named actions, link objects between applications. Can work while original UI remains elsewhere. Requires an observable/actionable contract, inferred or provided.
3. **Model-native composition:** Several independently authored views edit one shared model with stable identity, transactions and undo. Best semantics, but arbitrary pre-existing apps do not already participate in that model.

A compelling product can combine all three and be explicit about the mode of each component. It should not market level 1 as level 3. Agent-generated bridges can dramatically expand coverage; they cannot infer information that is absent from every available observation or add atomicity to systems that lack it.

## Evidence that matters

- **MCP Apps is a cooperative UI transport, not arbitrary extraction.** Servers declare HTML UI resources, hosts place them in sandboxed iframes, and tool/UI communication passes through a host bridge. A third-party adapter can author such a UI around an existing app, but that is a newly authored control surface, not the original app's native panel. [MCP Apps overview](https://apps.extensions.modelcontextprotocol.io/api/documents/overview.html)
- **CDP is a powerful browser instrumentation channel.** Its DOM, Runtime and Accessibility domains expose rendered structure, execution and accessibility information. AX metadata includes selection, focus, control relationships and values; it is not a universal document schema. [CDP](https://chromedevtools.github.io/devtools-protocol/index.html), [Accessibility domain](https://chromedevtools.github.io/devtools-protocol/tot/Accessibility/)
- **Chrome extension isolation matters.** Content scripts share the page's DOM but have an isolated JavaScript environment; page JavaScript variables are not automatically available. Main-world instrumentation is a different, riskier bridge, not a consequence of reading the DOM. [Chrome content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts)
- **Observation support is conditional on the app.** Apple's AX observer API explicitly has an unsupported-notification error. Accessibility can therefore be excellent for some controls and require reconciliation polling or fail entirely for others. [AXObserverAddNotification](https://developer.apple.com/documentation/applicationservices/1462089-axobserveraddnotification)
- **Editors can expose the live buffer.** VS Code's extension API exposes text-document change events, dirty state, versions, editors and selections. A disk watcher alone observes saved files, not necessarily the document the user currently sees. LSP defines full/incremental document synchronization, but is an editor/server contract, not a universal way to attach to any editor. [VS Code API](https://code.visualstudio.com/api/references/vscode-api), [LSP specification](https://github.com/microsoft/language-server-protocol/blob/gh-pages/_specifications/lsp/3.18/specification.md)
- **Live has a useful, bounded semantic model.** Max for Live's live.observer emits observed property changes; not all properties are observable, notifications cannot directly trigger Live Set modifications, and the API runs on the main thread with deferred messages. Persistent mapping can preserve object associations. These are the sorts of actual lifecycle and threading constraints every bridge must encode. [live.observer](https://docs.cycling74.com/reference/live.observer)
- **Do not repeat the outdated “Link cannot transmit audio” claim.** Current Ableton documentation includes Link Audio for real-time audio streaming between compatible peers. Beat/tempo/phase synchronization and audio transport are still distinct from sharing clips, arbitrary plugin state or project structure. [Ableton synchronization manual](https://www.ableton.com/en/manual/synchronizing-with-link-tempo-follower-and-midi/)
- **Local-first schema research is directly relevant but conditional.** Cambria demonstrates bidirectional schema lenses and also explains their limits: sufficiently different representations force tradeoffs, and some mappings have no unique reverse. Its first-name/last-name versus full-name example is a useful miniature of the universal-adapter problem. [Project Cambria](https://www.inkandswitch.com/cambria/)
- **CRDTs converge representations; they do not settle domain intent.** Automerge preserves divergent same-property values and chooses a deterministic visible winner. A converged document can still encode the wrong business or creative decision. [Automerge conflicts](https://automerge.org/docs/reference/documents/conflicts/)
- **Passthrough mods are a useful analogy, not proof of universal support.** A current community guide uses the term for two running games linked together, distinguishing it from engine rewrites and compositing; its SkyCraft example uses a Skyrim script-extender plugin and Minecraft Fabric mod. This is evidence for application-specific bridges and ownership boundaries, not “no adapters required.” Treat the guide's breadth claims as project claims, not independently benchmarked capability. [AI game-modding guides](https://github.com/trevaintdead/ai-game-modding-guides)

## Proposed architecture: a semantic sidecar, not a second source of truth everywhere

The following is design analysis, not a claim that an existing product implements it.

### 1. Per-app adapters publish contracts

Each adapter declares capabilities separately:

- snapshot reads: which objects and properties are available, and whether saved, live or rendered
- subscriptions: which changes are complete, best-effort, coalesced, polled or unavailable
- actions: typed parameters, units, preconditions, expected effects, reversibility and idempotency
- identity: stable source IDs versus session-only handles, reconnect strategy and object-lifetime rules
- fidelity: exact domain value, exact displayed value, inferred value, or opaque visual region
- permissions: applications, documents, origins, actions and data fields that the adapter may access
- supported versions and runtime fingerprints; explicit degraded/disconnected states

The adapter contract should distinguish “read current selection,” “edit document range,” “set visible text field” and “send keyboard input.” These have radically different guarantees even if one demo produces the same pixels.

### 2. Prefer the strongest available interface, not one universal interface

A practical ordering is existing domain APIs/extensions/document protocols; OS automation/accessibility; browser DOM/CDP instrumentation; file exchange; pixel/input fallback. The order is not absolute: a rich native accessibility control can outperform a thin API, and files can be the correct authority for saved artifacts.

MCP is a useful envelope for exposing available tools/resources, with version-negotiated change notifications where supported. It does not create source-app subscriptions, stable identity, transactions or complete semantic coverage by itself. Pin the actual protocol and adapter capability versions rather than assuming all MCP hosts implement the same current behavior.

An agent can inspect permitted interfaces, propose a bridge, generate selectors/transforms/tests, and repair version drift. Promote generated bridges through read-only discovery, sandboxed replay, explicit contract tests and user-approved write capabilities. Ordinary steady-state input should execute deterministic bridge code, not ask an LLM to reinterpret the whole screen every frame.

### 3. A typed graph connects objects, not just copied text

A reference needs at least application/provider, account/workspace, document ID, object ID, revision, and session/epoch where relevant. Keep file paths, browser tabs, window handles and display names as locators, not unquestioned identities. A renamed document may be the same object; a reopened tab may be a different session; two “Untitled” files are not interchangeable.

Represent links with typed intent: derived preview, citation of a snapshot, follows selection, mirrors a parameter, or proposes an edit. Keep original source references and transformations. Avoid a monolithic schema that reduces a rich document or musical project to lowest-common-denominator JSON. Preserve source-native opaque fields where possible and declare loss when crossing models.

### 4. Separate four state classes

- **Durable source state:** saved document/project and remotely committed state
- **Live uncommitted state:** unsaved editor buffer, staged form edits, in-flight collaboration changes
- **Ephemeral interaction state:** focus, selection, pointer capture, viewport and transport position
- **Derived state:** PDF build, search results, citation metadata, waveform analysis

Every derived result records its exact input revision or content hash. A PDF built from an older buffer remains usable but must be visibly stale. Focus and pointer state should not be event-sourced as if they were durable user intent. Selection can be shared intentionally without making every click in one pane yank focus elsewhere.

### 5. Changes flow as observations and commands

The host records an observation with source, source revision or sequence, timestamp, confidence/fidelity and originating operation ID. A binding transforms that observation into a proposed or authorized command. The command includes target identity, expected version, operation ID, precondition, desired effect and verification rule. The adapter reports accepted, committed, conflicted, failed, or outcome unknown.

Use subscriptions plus periodic bounded reconciliation, because sessions drop and notifications may be incomplete. Use sequence gaps and fresh snapshots to recover. Time of arrival is not causal order. Tag origin and transaction IDs to suppress echoes. Normalize values and compare semantic equality so rounding (for example 0.501 versus a displayed 50%) does not produce feedback loops. Cyclic graphs require explicit direction, damping or a declared fixed-point rule; do not allow “all fields are bidirectional” by default.

### 6. Choose ownership per field and operation

Default: the original app owns its domain state; the sidecar owns composition/layout, links and provenance. A mirror is a cache with a source version. For scalar controls use one active writer or explicit arbitration. For simultaneous text editing, use CRDT/OT only where both endpoints expose compatible operations, stable anchors and a feasible mapping.

Putting a CRDT in the host does not turn an opaque editor into a CRDT participant. Translating two whole-document snapshots can lose selection, formatting, hidden annotations and edit intent. A text-only round trip that deletes a footnote is failure even if all visible paragraphs match.

### 7. Transactions and undo are honest about boundaries

Within host-owned state, transactions and event sourcing are straightforward. Across independent apps, use a saga: checkpoint, perform steps, verify each, and compensate if supported. Do not promise an atomic cross-app transaction without prepare/commit support from every involved system. Keep an “outcome unknown” state after a transport timeout; blind retries can duplicate destructive or external effects.

The operation journal stores intent, before/after source versions, observed effects, transform version, permissions and compensating action. Replaying the journal must not automatically repeat external side effects. “Undo this composition action” should reverse the adapter's operation conditionally, preserve intervening user edits, and state partial undo honestly. Yjs provides transaction-origin-scoped selective undo inside its model; that does not reach into another application's private undo stack. [Yjs UndoManager](https://docs.yjs.dev/api/undo-manager)

For selections inside shared text, stable relative anchors can survive remote insertions; raw character offsets cannot. Yjs documents this distinction. Cross-app adapters must still define how those anchors map to their own models. [Yjs relative positions](https://docs.yjs.dev/api/relative-positions)

## Scenarios and sharp counterexamples

### LaTeX editor + PDF + browser research

Best slice: keep the real editor running; an extension exposes live text, version and selection. Compile an immutable snapshot and bind generated PDF plus synchronization data to that snapshot. Browser-selected quotations become attributed snippets carrying URL, capture time and selected text; inserting a citation is an explicit editor action. PDF clicks resolve to the corresponding source revision, and then transform the anchor through subsequent edits where possible.

SyncTEX's auxiliary data relates source locations to output geometry. It does not make PDF-to-LaTeX a general inverse transformation. Macros, duplicated rendered strings and generated content make arbitrary reverse edits ambiguous. [XeTeX SyncTEX implementation description](https://mirrors.ctan.org/info/knuth-pdf/xetex/xetex-changes.pdf)

Counterexamples: a disk-only bridge shows a saved paragraph while the editor contains a different unsaved one; a PDF click applies the old line number to the new buffer; inserting a quote assumes clipboard provenance; a PDF panel “edits” a formula that was produced by a macro in another file. These require buffer access, revision-aware anchors, captured provenance and explicit ambiguity handling respectively.

### Ableton + plugin + browser musical workflow

Useful composition: pin real plugin UI and Live controls; expose selected track/clip and supported parameters through Max for Live or an existing control interface; bring a browser sample/research result into an explicit import operation; link notes to a project and timestamp. The browser can display a custom macro knob whose semantic bridge changes a known Live parameter and verifies the resulting value.

Keep audio DSP and tight timing on the host's established audio/MIDI scheduling path. The orchestration graph should not put LLM inference, browser timers, accessibility polling or arbitrary IPC round trips in a sample-accurate control loop. Live's API thread limitations concretely show why semantic orchestration and real-time signal control need separate paths. [Live API overview](https://docs.cycling74.com/legacy/max7/vignettes/live_api_overview)

Counterexamples: the plugin visually exposes a knob but provides neither parameter identity nor unit/range mapping; a preset hides internal state not represented by automatable parameters; the same numeric value means normalized amplitude in one app and dB in another; changing a parameter starts recording automation; deleting/replacing a device invalidates its old identity. A cropped knob remains interactable but cannot truthfully become a fully typed generic instrument control without additional discovery.

### Documents and notes

Start with references and source-linked excerpts. Offer narrowly scoped write-back for well-understood blocks or ranges, with loss detection and a review diff. Keep source comments, suggestions, tables and IDs; never silently flatten them into Markdown and overwrite the original.

Google Docs is a good example of an API with meaningful concurrency: requiredRevisionId can reject stale writes; targetRevisionId can have the server merge against collaborator changes; a batch has documented atomic application. Those guarantees are local to the Docs operation, not the note app or the entire desktop workflow. [Docs batchUpdate](https://developers.google.com/workspace/docs/api/reference/rest/v1/documents/batchUpdate)

The current Docs documentation supports programmatic suggestions/comments, so outdated claims that these are categorically unavailable should be avoided. It also warns that comment/suggestion persistence can partially fail despite document-model changes; verification must inspect that status. [Docs comments and suggestions](https://developers.google.com/workspace/docs/api/how-tos/suggestions)

Counterexample: one pane shows “suggestions accepted” preview while another addresses positions in the actual document model. Both display plausible text but refer to different ranges. The bridge must specify view mode, revision and index semantics.

## What remains structural even with excellent AI and generous engineering effort

1. **Unobservability:** Two internal states may produce the same available pixels/AX tree/API output but require different edits. No inference can guarantee which is true without another observation channel.
2. **Non-invertibility:** Flattened rich text, compiled PDFs, rendered audio and lossy exports do not uniquely determine their source models. Preserve ambiguity or ask; do not invent an inverse.
3. **Authority:** A user may authorize local access but the OS, service, enterprise policy or account still denies a needed operation. The bridge cannot legitimately grant itself missing privileges.
4. **Atomicity and irreversibility:** Independent uncooperative apps cannot generally promise all-or-nothing multi-app changes, exactly-once delivery, or undo of an already sent external communication.
5. **Timing:** Faster semantic reasoning does not provide real-time scheduling guarantees or remove downstream latency and jitter.
6. **Absent semantics:** An accessibility slider's number does not identify its musical units, effect on automation, ownership, or safe update frequency.
7. **Hidden lifecycle:** Native control handles, DOM nodes and remote objects can be replaced. Persistence requires rediscovery and verified domain identity.

Other problems are hard but not fundamentally impossible: adapter generation, value mapping, visual-region tracking, reconnection, testing, maintaining a compatibility registry, and reconstructing some undocumented protocols. Grant the proposed system strong engineering on those; the seven boundaries above still matter.

## Security, trust and distribution implications

Keep credentials in their originating browser/app or an isolated credential broker. Do not turn a composed panel into permission for every adjacent component to read that app's session. Use per-origin processes, narrow named bridge capabilities, explicit data-flow grants and a visible permission inspector. Rendered text is untrusted data, never authority for installing a bridge or authorizing write-back. Generated adapter code needs sandboxing, review, provenance, version pins and revocation.

Chrome's security history makes this concrete: remote debugging can expose session data, and Chrome changed debugging switches for the default profile. Current Chrome also offers a consent-based connection to an active session with a permission dialog and visible automation banner; “just attach to every browser silently” is not a sound design assumption. [Debugging security change](https://developer.chrome.com/blog/remote-debugging-port), [active-session connection](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session)

A feature can be technically feasible but unsuitable to distribute because it depends on credential copying, bypassing protection, prohibited automation, unlicensed embedded assets, or fragile private APIs. Licensing, platform distribution terms, app-specific terms and anti-circumvention questions require qualified review for the exact implementation/jurisdiction. This is a risk checklist, not a legal conclusion. User-owned local data and code-only adapters reduce some practical exposure but do not settle every rights question.

## Proof-of-concept acceptance tests

Build one vertical workflow, ideally editor + PDF + browser, before claiming arbitrary desktop composition. These are proposed targets, not measurements:

- Both original and composed views remain live. Ten edits made in either supported editing surface arrive at the other without losing unsaved work; declare actual editing surfaces versus visual portals.
- Source-to-view latency: report median/p95 separately for subscription, polling and pixel adapters. A sensible initial local semantic target is p95 under 100 ms excluding compilation/network operations; do not call all adapters real-time if only one meets it.
- Version integrity: force slow compilation while typing; never label an older PDF as current. Verify a PDF click selects the intended source span or explicitly reports ambiguity.
- Concurrency: edit the same field from source and shell; demonstrate deterministic documented policy or a surfaced conflict, no silent overwrite.
- Loop control: connect A→B→A with normalization differences; 1,000 updates must quiesce without storms or oscillation.
- Failure recovery: kill adapter after command acceptance but before acknowledgement; restart without blind duplicate action. Show verified committed/failed/unknown outcomes.
- Identity: rename/reorder/reopen documents or tracks and duplicate names; links follow the intended object or stop safely.
- Undo: compound action across two apps; inject failure in second step and a concurrent user edit in first. Demonstrate compensation preserves that edit, or present partial undo clearly.
- Fidelity: a rich document containing comments, footnotes, suggestions and tables must round-trip unchanged outside the explicitly edited field; otherwise write-back is refused or visibly lossy.
- Least privilege: a malicious source page requesting data from another app cannot obtain it or authorize a new binding. Revocation immediately stops new reads/writes.
- Adapter maintenance: run the same tests after a source app update and DOM/control replacement. Record breakage rate, repair time and false-success rate, not only generation speed.
- Music follow-on: parameter changes work both ways and remain correct after preset/device changes. Measure timing separately; verify that semantic traffic never blocks audio processing.

## Product judgment under the generous-difficulty assumption

The valuable unit is a reusable workflow fragment with a declared contract: “this is the live current selection; this control changes that parameter; this preview is from that revision.” Agents could lower the cost of creating and repairing such fragments enough to make a new desktop layer worthwhile. The defensible work would include adapter certification, identity/provenance, failure recovery and safe composition semantics, not merely generating a nicer dashboard.

A credible promise is “assemble live tools and connect their supported meanings, with graceful opaque-surface fallback.” “Every app becomes arbitrary interoperable parts, with universal shared state and undo” exceeds what source-independent observation and independent application authorities can guarantee.
