# Semantics: evidence and documented mechanisms

Editorial extraction, 2026-10-07. Source research was read-only; no listed software was installed, executed, or benchmarked. Documented APIs, author-reported demonstrations, source inspections, and untested compatibility are distinguished below. Source-level conclusions are limited to the investigated interfaces and versions. No implementation architecture is prescribed here.

- **MCP Apps is a cooperative UI transport, not arbitrary extraction.** Servers declare HTML UI resources, hosts place them in sandboxed iframes, and tool/UI communication passes through a host bridge. A third-party adapter can author such a UI around an existing app, but that is a newly authored control surface, not the original app's native panel. [MCP Apps overview](https://apps.extensions.modelcontextprotocol.io/api/documents/overview.html)
- **CDP is a powerful browser instrumentation channel.** Its DOM, Runtime and Accessibility domains expose rendered structure, execution and accessibility information. AX metadata includes selection, focus, control relationships and values; it is not a universal document schema. [CDP](https://chromedevtools.github.io/devtools-protocol/index.html), [Accessibility domain](https://chromedevtools.github.io/devtools-protocol/tot/Accessibility/)
- **Chrome extension isolation matters.** Content scripts share the page's DOM but have an isolated JavaScript environment; page JavaScript variables are not automatically available. Main-world instrumentation is a different, riskier bridge, not a consequence of reading the DOM. [Chrome content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts)
- **Observation support is conditional on the app.** Apple's AX observer API explicitly has an unsupported-notification error. Notification coverage depends on the target control/application. [AXObserverAddNotification](https://developer.apple.com/documentation/applicationservices/1462089-axobserveraddnotification)
- **Editors can expose the live buffer.** VS Code's extension API exposes text-document change events, dirty state, versions, editors and selections. A disk watcher alone observes saved files, not necessarily the document the user currently sees. LSP defines full/incremental document synchronization, but is an editor/server contract, not a universal way to attach to any editor. [VS Code API](https://code.visualstudio.com/api/references/vscode-api), [LSP specification](https://github.com/microsoft/language-server-protocol/blob/gh-pages/_specifications/lsp/3.18/specification.md)
- **Live object observation.** Max for Live's live.observer emits observed property changes; not all properties are observable, notifications cannot directly trigger Live Set modifications, and the API runs on the main thread with deferred messages. Persistent mapping can preserve object associations. [live.observer](https://docs.cycling74.com/reference/live.observer)
- **Correction: Link Audio.** Current Ableton documentation includes Link Audio for real-time audio streaming between compatible peers. Beat/tempo/phase synchronization and audio transport are still distinct from sharing clips, arbitrary plugin state or project structure. [Ableton synchronization manual](https://www.ableton.com/en/manual/synchronizing-with-link-tempo-follower-and-midi/)
- **Bidirectional schema lenses.** Cambria demonstrates bidirectional schema lenses and also explains their limits: sufficiently different representations force tradeoffs, and some mappings have no unique reverse. Its examples include first-name/last-name versus full-name mappings. [Project Cambria](https://www.inkandswitch.com/cambria/)
- **CRDTs converge representations; they do not settle domain intent.** Automerge preserves divergent same-property values and chooses a deterministic visible winner. A converged document can still encode the wrong business or creative decision. [Automerge conflicts](https://automerge.org/docs/reference/documents/conflicts/)
- **Passthrough terminology.** A current community guide uses the term for two running games linked together, distinguishing it from engine rewrites and compositing; its SkyCraft example uses a Skyrim script-extender plugin and Minecraft Fabric mod. This is evidence for application-specific bridges and ownership boundaries, not “no adapters required.” The guide's breadth claims were not independently benchmarked. [AI game-modding guides](https://github.com/trevaintdead/ai-game-modding-guides)

<!-- Origin: semantics.md lines 19–28 -->

Yjs provides transaction-origin-scoped selective undo inside its own model. This API does not itself access a different application's private undo stack. [Yjs UndoManager](https://docs.yjs.dev/api/undo-manager)

<!-- Origin: semantics.md lines 87–87 -->

For selections inside shared text, stable relative anchors can survive remote insertions; raw character offsets cannot. Yjs documents this distinction. Its documented behavior concerns positions within the Yjs model. [Yjs relative positions](https://docs.yjs.dev/api/relative-positions)

<!-- Origin: semantics.md lines 89–89 -->

SyncTEX's auxiliary data relates source locations to output geometry. It does not make PDF-to-LaTeX a general inverse transformation. Macros, duplicated rendered strings and generated content make arbitrary reverse edits ambiguous. [XeTeX SyncTEX implementation description](https://mirrors.ctan.org/info/knuth-pdf/xetex/xetex-changes.pdf)

<!-- Origin: semantics.md lines 97–97 -->

Counterexamples: a disk-only bridge shows a saved paragraph while the editor contains a different unsaved one; a PDF click applies the old line number to the new buffer; inserting a quote assumes clipboard provenance; a PDF panel “edits” a formula that was produced by a macro in another file. These are failure scenarios, not measured incidents.

<!-- Origin: semantics.md lines 99–99 -->

Counterexamples: the plugin visually exposes a knob but provides neither parameter identity nor unit/range mapping; a preset hides internal state not represented by automatable parameters; the same numeric value means normalized amplitude in one app and dB in another; changing a parameter starts recording automation; deleting/replacing a device invalidates its old identity. A cropped knob remains interactable but cannot truthfully become a fully typed generic instrument control without additional discovery.

<!-- Origin: semantics.md lines 107–107 -->

Google Docs documents concurrency controls: requiredRevisionId can reject stale writes; targetRevisionId can have the server merge against collaborator changes; a batch has documented atomic application. Those guarantees are local to the Docs operation, not the note app or the entire desktop workflow. [Docs batchUpdate](https://developers.google.com/workspace/docs/api/reference/rest/v1/documents/batchUpdate)

<!-- Origin: semantics.md lines 113–113 -->

The current Docs documentation supports programmatic suggestions/comments. It also warns that comment/suggestion persistence can partially fail despite document-model changes. [Docs comments and suggestions](https://developers.google.com/workspace/docs/api/how-tos/suggestions)

<!-- Origin: semantics.md lines 115–115 -->

Counterexample: one pane shows “suggestions accepted” preview while another addresses positions in the actual document model. Both display plausible text but refer to different ranges.

<!-- Origin: semantics.md lines 117–117 -->

1. **Unobservability:** Two internal states indistinguishable through every permitted observation channel available to an implementation cannot be distinguished reliably by that implementation. A blank AX tree alone does not establish this condition; another permitted runtime channel may expose more.
2. **Non-invertibility:** Flattened rich text, compiled PDFs, rendered audio and lossy exports do not uniquely determine their source models.
3. **Authority:** A user may authorize local access but the OS, service, enterprise policy or account still denies a needed operation. The bridge cannot legitimately grant itself missing privileges.
4. **Atomicity and irreversibility:** Independent uncooperative apps cannot generally promise all-or-nothing multi-app changes, exactly-once delivery, or undo of an already sent external communication.
5. **Timing:** Faster semantic reasoning does not provide real-time scheduling guarantees or remove downstream latency and jitter.
6. **Absent semantics:** An accessibility slider's number does not identify its musical units, effect on automation, ownership, or safe update frequency.
7. **Hidden lifecycle:** Native control handles, DOM nodes and remote objects can be replaced.

<!-- Origin: semantics.md lines 121–127 -->

Chrome's security history makes this concrete: remote debugging can expose session data, and Chrome changed debugging switches for the default profile. Current Chrome also offers a consent-based connection to an active session with a permission dialog and visible automation banner. [Debugging security change](https://developer.chrome.com/blog/remote-debugging-port), [active-session connection](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session)

<!-- Origin: semantics.md lines 135–135 -->
