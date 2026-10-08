# Eversion: an owned runtime for composing live capabilities

Independent Astra architecture recommendation — 7 October 2026. This is a design proposal, not an implementation or compatibility certification. The approved intent, brief and all seven findings were read in full after hash verification; see [input verification](input-verification.md). No recommendations, other architect's work, or prior report/history informed this draft. The accompanying research memos contain primary-source checks and alternative mechanisms.

## 1. The architectural decision

Build Eversion as a **native macOS composition runtime with its own capability protocol, workflow language, source supervisors, scene compositor and agent development environment**. Existing applications remain running engines. Their useful functions become addressable, observable and controllable participants in programs that Eversion owns.

The unit of composition is a *live capability attached to an entity and a source session*, not an application rectangle. A surface is one representation of a capability. A capability can also have typed observations, commands, time streams and persistence guarantees. A workflow supplies relationships among those capabilities and changes their behavior over time. It can create new participants, change which tool owns a task, maintain parallel alternatives, wait days for evidence, enlist an agent, and reconfigure its interface while the underlying work continues.

Own the core mechanisms rather than making a permanent chain of automation products. Use platform frameworks and browser-engine code as machinery beneath explicit Eversion contracts. The difficult, distinctive work is the contract boundary: connecting a real source's state, identity, operations, interaction context and lifecycle to a program that the user can reshape.

There must be two permanent execution modes:

- **Attached execution:** use the person's already-running native app or authorized browser session. This retains their actual document, account, unsaved work and established application environment. Eversion owns observation, mediation and workflow behavior, but cannot assume ownership of the source runtime.
- **Managed execution:** Eversion launches and supervises a compatible browser context, instrumented application instance, or isolated macOS guest/session. It can own more of rendering, scheduling, input and recovery. The new execution context is not secretly the same state as an existing app.

Neither mode is an inferior stepping stone. Some workflows require the current authenticated tab; others justify a dedicated, deeply controllable engine. A composition can combine both. Support is a matrix of guarantees per operation, not a ladder in which pixels inevitably become a complete model.

The product claim should be: **compose the capabilities we can establish in your actual tools, and let agents help establish more**. No predetermined access to the stack is required to begin discovery. It does not follow that an unknown app can always supply every requested capability.

## 2. What the evidence establishes—and what it does not

The approved packet supports a much wider design space than window cropping. It also supplies several counterexamples to careless generalization.

| Evidence | Consequence for this architecture |
|---|---|
| ScreenCaptureKit exports rendered frames; AX exposes implemented attributes/actions/notifications | Broad native attachment is possible without source access. Captured pixels and AX references are separate channels, and neither is a complete document model. |
| Runtime interception and framework probes can inspect behavior beneath the UI | Stack-specific discovery deserves a first-class architecture, rather than being dismissed as impossible or treated as a universal solution. |
| Browser execution, DOM, paint, origin and framework ownership are different boundaries | Retain the original runtime closure and project presentation; add semantic adapters where verified. |
| Real passthrough mods bridge events, identities, state and rendering between still-running engines | The useful analogy is negotiated engine authority and explicit bilateral contracts, not flattening everything into a common screenshot. |
| Existing crops share source selection, scrolling and modal state | Independent views require independent source views, instances, or host-owned semantic representations. |
| Lossy representations and remote hidden state exist | Unknown facts and unsupported inverse operations must remain representable inside the workflow. |

These are findings from documented interfaces and author-reported systems, not measurements on the user's Mac. The source packet explicitly separates SkyCraft's draft design from its inspected protocol header; this architecture does not treat that draft as proof of a GPU-sharing implementation. WinCuts demonstrates interactive local regions, not universal semantic extraction. [WinCuts paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2004/01/WinCuts.pdf), [SkyCraft protocol](https://github.com/chasmlol/SkyCraft/blob/main/protocol/skycraft_protocol.h).

The actual information boundary is precise: if two source states are indistinguishable through **all permitted channels available to the implementation**, Eversion cannot reliably choose different behavior based on that distinction. A blank AX tree is not such a proof; a permitted runtime probe may reveal more. Likewise, engineering difficulty is not a permission boundary, and owning more code does not grant authority over remote servers or protected processes.

## 3. The model Eversion owns

### 3.1 Five objects, deliberately kept separate

1. **Source:** a particular app/browser/guest execution and its authority context. It includes process or renderer incarnation, account identity when observable, document identities, adapter version and granted access. A PID or URL alone is insufficient.
2. **Entity:** a thing the workflow cares about: a paragraph, dataset, research claim, clip, cue, supplier, issue or proposed decision. An entity has source-native identities and explicitly asserted cross-source relationships. Eversion owns its cross-tool identity, not necessarily the underlying content.
3. **Capability:** a bounded promise to observe, act, render, allocate a view or exchange a timed stream in a source. Its type includes preconditions, available evidence and resource requirements.
4. **Case:** a durable running instance of a workflow over entities. Cases have state, outstanding work, child cases, permissions, proposals and effect receipts. Cases can outlive every visible window.
5. **Presentation:** a view of entities and cases assembled from original live surfaces, Eversion-native controls, derived data, proposal editors and explicit snapshots. Closing a presentation need not cancel the case.

A universal domain schema would be a mistake. Eversion needs a small common vocabulary for identity, version, causality, capability, relationship and effects; application-specific schemas remain namespaced. `Live.Clip`, `Editor.TextRange` and `Review.Comment` should not be forced into a generic object whose writable fields erase their semantics.

### 3.2 Observations are evidence, not omniscience

Every observation carries:

```
subject: source-qualified identity + incarnation
value: typed payload or explicit unknown
sourceRevision: native revision, sequence, or unavailable
observedAt: monotonic time + clock/boot identity + broker receipt time
validity: live | stale | missing | conflicted | invalidated
basis: documented contract | measured mapping | user assertion | inference
provenance: source channel + adapter version + derivation dependencies
```

Clock domains are explicit: raw guest/source/host monotonic timestamps are not directly comparable. Reboot, source restart and clock discontinuity require a new freshness assessment. Persisted calendar deadlines are separate from elapsed-time expiry.

A numerical model confidence is never a substitute for identity or authority. An inferred correspondence can be useful in a recommendation while being inadmissible for an unattended write. Provenance follows derived values: a fresh spreadsheet formula does not make its stale input fresh.

There is no assumed global snapshot of independent apps. The kernel records a frontier of source revisions and observation times. Each join states whether it requires exact revision correspondence, a maximum observation skew, a common artifact build, or an explicitly provisional answer. A figure and paragraph generated from different dataset revisions should be visibly inconsistent even if each source looks healthy.

Stable references use the strongest available address: vendor object ID and revision, application document ID, framework model key, durable text anchor, then constrained semantic/visual locator. Text offsets are valid only against a stated revision. Yjs relative positions are an example of a stronger anchor *inside a cooperating Yjs document*, not a portable fix for arbitrary editors. [Yjs relative positions](https://docs.yjs.dev/api/relative-positions).

Rebinding after relaunch is a resolution operation with evidence. Ambiguous duplicate labels suspend affected writes. Where deletion/recreation is observable, it creates a new incarnation even if the label and screen position are identical. Where continuity cannot be established after a gap, capabilities requiring stable identity suspend rather than guessing. Observed account changes revoke session-bound capabilities; unobservable account state cannot satisfy an account-specific guarantee.

### 3.3 Authority is assigned per fact and operation

A source app owns its actual document and business rules. Eversion owns relationships it introduces, agent proposals, workflow state, composition definitions and host-created data. A mapping states whether it is an observation, a one-way derivation, a proposed edit or a supported bidirectional edit lens.

Do not connect two writable fields with an unrestricted “sync” edge. A link must state direction, transformation, conflict policy and which side may originate changes. Conversion from many assignees to one assignee cannot preserve every intended edit. Cambria's concrete example illustrates that some interoperability choices are policy decisions, not missing serialization code. [Cambria](https://www.inkandswitch.com/cambria/).

Two-way operation is admitted only with a declared consistency relation and explicit treatment of nonrepresentable values. Otherwise use a proposal: “update this title to match that title.” Sources may reject it, and that rejection is meaningful workflow state.

## 4. Capability contracts and source supervision

### 4.1 The protocol

Define an Eversion protocol rather than exposing arbitrary scripts as the basic component interface. Transports may include local IPC, target-side plugins, a browser bridge and guest channels. A capability advertises the following facets independently:

| Facet | Required declaration |
|---|---|
| Identity | Source session, object resolver, incarnation rules, account/document constraints |
| Observation | Schema, completeness, subscription/polling behavior, revision semantics, freshness and failure signals |
| Action | Argument types/units, preconditions, authority, expected effects, thread affinity, reentrancy constraints |
| Confirmation | What proves dispatch, local acceptance, resulting state, and durable remote completion |
| Presentation | Surface or host-view provider, geometry, accessibility, input envelope, transient surfaces |
| Multiplicity | Shared source view, allocatable independent view, separate instance, or read-only snapshot |
| Resources | Input/focus/modal/selection/document-write/real-time resources that must be leased |
| Recovery | Reconnect, re-resolve, snapshot/restore, cancellation and compensations actually supported |

One source can expose several implementations of a function. The planner chooses based on required guarantees and permitted access: a documented revision-checked edit, a source-native command, a DOM interaction, or a supervised gesture. It must not silently substitute “click the apparent button” for a command whose contract requires a durable remote receipt.

A **contract record** accompanies each adapter: supported source fingerprints, documented facts, measured conformance cases, remaining assumptions and invalidation triggers. This is engineering evidence, not a formal proof that the app will never change. Adapter upgrades produce new contract versions; old workflows retain their old meaning until an explicit compatible migration is accepted.

### 4.2 Leases name the actual shared resource

Several portals can display the same source at once. They do not necessarily possess several independent input contexts. Lease the resource at its real scope: process keyboard seat, window first responder, document selection, modal chain, drag session, source transaction or plugin transport.

The broker can parallelize genuinely independent semantic operations while serializing only conflicting resources. It must not impose a single global queue merely because one fallback uses GUI events. Conversely, two source windows can still share application-level modal state. Adapter declarations start conservative, and conformance tests justify narrower conflicts.

A lease controls Eversion's actors; it cannot stop a human or another program changing an attached app. Native comparison followed by a GUI write usually has a race unless the source supplies an atomic conditional operation. For an operation that requires strict revision safety, such a backend is ineligible. For a reversible supervised edit, the UI may permit a weaker contract with immediate verification. These are different capabilities, not confidence settings on the same promise.

Acquire required resource sets atomically or in a fixed global order; never hold one ordinary interaction lease indefinitely while waiting for another. Release ordinary leases before waiting for agents, network outcomes or human decisions. Bound queued ownership, make pending acquisition cancelable, and identify non-preemptible portions of gestures so cancellation ends them coherently. Test contention and cyclic workflows explicitly.

The human can always reclaim interaction. Reclamation cancels queued gestures, ends a drag/marked-text session correctly where possible, and suspends affected agent work. It does not pretend to undo effects already issued.

### 4.3 Process layout on macOS

Use a native AppKit shell and Metal/Core Animation presentation engine. Keep the composition kernel, source supervision, agent execution and high-bandwidth rendering on separate failure boundaries. A suitable implementation is a memory-safe systems-language kernel with a small native bridge; the exact language is subordinate to the contracts.

```
Native shell / accessible composition views
             | presentation intents
Composition kernel — entity graph, derivations, case machines, effect ledger
             | typed grants, observations, commands
Source brokers — native attachment | browser attachment | managed runtimes
             |                                   |
Target-side adapters                         Render streams / input bridges

Agent workbench -> proposed adapter/program changes -> validator -> kernel
```

Generated transforms run in a restricted interpreter or WebAssembly compartment with typed imports, resource quotas and no ambient OS access. Native probes are a different trust class because they run inside source processes; a sandbox around the agent does not sandbox injected code.

The main distribution should be a directly distributed, signed and notarized macOS application with narrowly scoped helpers. Apple's documented App Sandbox restrictions on assistive AX usage and arbitrary Apple Events make the Mac App Store sandbox an unsuitable assumption for the full attachment product. Signing/notarization does not erase TCC or target protections. The architecture may sandbox individual helpers where their functions permit it. [Apple App Sandbox restrictions](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox), [Apple DTS clarification](https://developer.apple.com/forums/thread/789663).

Capture, Accessibility and Apple-event automation remain separate grants. The user's permission to view a surface need not grant an agent permission to read its contents, and neither implies permission to send data elsewhere. The OS grants may be coarse; Eversion's own broker attenuates them to particular sources, entities, operations and destinations. This is an internal security boundary, not a claim that the OS enforces each of those finer scopes.

## 5. Distinct solutions for distinct source classes

### 5.1 Native attachment: preserve the current app, expose what is real

Implement direct ScreenCaptureKit, AX, scripting and event backends rather than depending on a general automation app as the permanent execution substrate. Correlate channels under one source identity. Use semantic actions for operations they actually represent; retain original surfaces for opaque interactions.

ScreenCaptureKit provides a native pixel path and frame metadata. A single-window stream captures the window; its `sourceRect` does not turn it into an element capture primitive. Deduplicate capture per source window and crop in Eversion's GPU compositor. Frames that are idle, suspended, blank or stopped must be distinguished from an actively updating surface. Occlusion, minimization, hiding and Spaces need separate measurements. [Apple capture sample](https://developer.apple.com/documentation/screencapturekit/capturing-screen-content-in-macos), [sourceRect](https://developer.apple.com/documentation/screencapturekit/scstreamconfiguration/sourcerect), [frame status](https://developer.apple.com/documentation/screencapturekit/scframestatus).

AX actions, setters and notifications can build useful native Eversion controls without reproducing every source click. Unsupported notifications trigger a declared polling strategy or loss of continuous observation, not invented events. Targeted keyboard delivery exists, but reaching a PID does not establish an independent responder, menu or text-input session. [AX notifications](https://developer.apple.com/documentation/applicationservices/1462089-axobserveraddnotification), [AX keyboard events](https://developer.apple.com/documentation/applicationservices/1462057-axuielementpostkeyboardevent), [AppKit event architecture](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/EventOverview/EventArchitecture/EventArchitecture.html).

When the required interaction exceeds the bridge contract, temporarily expose the complete original app interaction. That is a deliberate workflow transition: preserve the case and return to it after the source supplies a verifiable result. It is preferable to a deceptively interactive crop that loses the save panel or input-method candidate window.

### 5.2 Native framework probes: expose behavior before moving views

Build a maintained family of **target-side capability probes**. Identify framework/runtime and compatible build, attach only through an admitted mechanism, inspect object/model relationships, observe actions and notifications, and export narrow commands. Objective-C runtime interception and Qt object inspection are concrete mechanisms; app-specific semantic discovery still remains. Frida and GammaRay are research references and possible lab tools, not an architecture that assumes their general admission into protected Mac apps. [Frida API](https://frida.re/docs/javascript-api/), [GammaRay launcher](https://docs.kdab.com/gammaray-manual/latest/gammaray-launcher-gui.html).

For an AppKit source, the first deep result should be something like `selectedAsset`, `renameAsset` and `observeDocumentRevision`. It need not be a transplanted toolbar. Preserve the original view hierarchy where practical. An advanced adapter may create a second source-owned view bound to the same model, or export a target-side rendering surface with explicit input dispatch. Simply detaching an NSView is not a generic preservation strategy for responder, constraints, delegates, undo and document ownership.

Framework-specific rendering can be much stronger than capture. Qt Quick's QQuickRenderControl explicitly gives a host control of an offscreen render loop and event handling, including Metal integration. It establishes an actual path for a Qt adapter; it does not establish that an arbitrary running Qt app can be retrofitted without reconstructing its view/model relationships. [QQuickRenderControl](https://doc.qt.io/qt-6/qquickrendercontrol.html).

SwiftUI and custom GPU apps require their own models. An NSHostingView is not evidence of a stable public SwiftUI state graph. AppKit bitmap caching is a snapshot facility, not proof of live export of every GPU/video/system effect. [NSView caching](https://developer.apple.com/documentation/appkit/nsview/cachedisplay(in:to:)).

The long-range native research program can include binary instrumentation, command interception and compatible managed copies. Treat an altered or re-signed app as a different execution identity, with compatibility experiments for authentication, entitlements, updates and document behavior. Library validation is controlled by the target; Eversion's entitlements do not authorize injection into another vendor's process. No ordinary installation should depend on changing the user's system security settings. That is a product choice; the technical research can still study deeper control in a separately authorized lab. [Apple library validation](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.cs.disable-library-validation).

### 5.3 Browser attachment: the existing session remains valuable

Use a consented extension/debugging bridge to an actual browser tab for DOM/runtime/AX observations, narrow commands and capture. Retain frame and document epochs, origin, account context and navigation identity. An isolated-world script can inspect DOM; access to page runtime state is a distinct bridge and remains untrusted input to Eversion.

Do not try to migrate arbitrary ordinary-browser profiles into Eversion. Current Chrome documents both default-profile restrictions on startup debugging switches and a consent-based route to an active browser session. These are different connection mechanisms. Their existence favors a first-class attachment backend rather than assuming that all authenticated sites will move into an embedded engine. [Chrome debugging change](https://developer.chrome.com/blog/remote-debugging-port), [active-session attachment](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session).

A browser page can lie in its DOM or model, and MAIN-world bridge messages can be influenced by page code. Such observations may inform the workflow; they cannot mint OS privileges, approve effects, or change the agent's instructions. Cross-origin and destination authority are enforced in the privileged broker, not entrusted to injected page code.

### 5.4 Managed browser: an engine-level projection API

This is the most promising place to own substantially more than automation plumbing. Build a native browser execution service, initially using CEF to test GPU and input integration, with an explicit path to an Eversion-maintained Chromium/Blink/Viz projection layer. The end-state is not an Electron dashboard or a reverse proxy with security headers removed.

The proposed API attaches a **projection root** to a source object and exports:

- selected paint output, geometric transforms, clipping, color and damage;
- source-coordinate hit-testing and frame generation;
- the corresponding accessibility subtree and navigation mapping;
- focus, pointer capture, marked-text and caret geometry;
- related transient surfaces and lifecycle invalidation;
- semantic capability handles where an adapter has established them.

The original page retains its document, JavaScript realm, framework ancestry, origins, workers, storage, network session and model. Initially retain the entire execution closure, not an allegedly minimal component dependency graph. A chart may keep its whole dashboard alive while only its useful presentation is shown.

Chromium's architecture separates paint/compositor structures and surface aggregation, providing concrete insertion points. It does not already implement arbitrary DOM projection with preserved behavior. DOM boundaries, stacking contexts, clip/effect trees and paint ownership differ; an engine fork must handle them deliberately. [Chromium compositor architecture](https://chromium.googlesource.com/chromium/src/+/main/docs/how_cc_works.md).

A projection also declares a visual-context policy: preserve pixels composed against the original backdrop, expand the visual dependency closure, or intentionally recompose against a new backdrop. Blend modes, masks, ancestor effects and overlapping siblings can make a subtree's appearance inseparable from its original surroundings; retaining execution alone does not solve that. [W3C compositing specification](https://www.w3.org/TR/compositing-1/).

This is an **engineering hypothesis with a demanding conformance test**, not a documented standard API. Keep source coordinates canonical to page code; transform presentation and native input outside that coordinate system. A new visual position must not silently change a component's framework ancestry. Menus rendered through a framework portal may live outside the selected DOM subtree, so the interaction envelope includes related transients. [React portal semantics](https://react.dev/reference/react-dom/createPortal).

Several projections of one document share its source selection and state. Giving one arbitrary page two contradictory viewport sizes or independent focused inputs would change its semantics. For independent search, scroll or responsive layout, allocate separate page instances or a verified application-specific second view. Sharing a browser storage context is not cloning a live heap or unsaved form. An application-aware branch is different again.

Load each managed application in its original top-level context and retain its nested frame structure and origins. Project selected descendants without promoting a cross-origin iframe into a different browsing context; storage partitioning, permissions and ancestor relationships must remain those of the source. Compose in the privileged native scene. A sees only A's principal, even when A and B appear adjacent. Browser source code and process isolation remain machinery to preserve, not inconveniences to disable. Chromium's OOPIF design is a useful architectural precedent for combining rendering, input and accessibility from different processes while retaining origin boundaries. [OOPIF design](https://www.chromium.org/developers/design-documents/oop-iframes/).

Authentication is a critical qualification. Google explicitly forbids OAuth authorization in developer-controlled embedded user agents of the relevant kind. An external authorization flow for Eversion's own API client does not transplant a third-party website session into the owned engine. Test accepted authentication per application, and keep browser attachment available wherever managed execution cannot establish a supported session. [Google OAuth policy](https://developers.google.com/identity/protocols/oauth2/policies), [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252).

### 5.5 Isolated execution: independence by allocating another engine

When a capability genuinely requires independent modal/focus/session state, allocate it rather than pretending a visual duplicate supplies it. Possible units are an app-provided document view, another compatible app instance, another browser context, or a separate macOS guest. Select the smallest unit that actually isolates the required resource.

Use Apple's Virtualization framework as a candidate for managed macOS guests on Apple silicon. A guest supplies an independent desktop input domain; it does not grant arbitrary injection privileges or duplicate remote-account state. Current Apple documentation supports iCloud for qualifying macOS 15+ guests with identity conditions, so blanket claims that all guests lack authentication are wrong. Transparent cloning of a logged-in identity is also not promised. [macOS virtualization](https://developer.apple.com/documentation/virtualization/virtualize-macos-on-a-mac), [iCloud in macOS guests](https://developer.apple.com/documentation/virtualization/using-icloud-with-macos-virtual-machines).

Prefer one guest per interaction domain, such as an editor and its plugins, not one guest per crop. Remoting must carry complete text input, clipboard/drag negotiation, file authority, accessible content and audio where required. A virtual display in the user's current login session only provides staging space; it does not create another keyboard seat or clone document state. Private virtual-display/WindowServer mechanisms are optional research backends, not foundational guarantees.

### 5.6 Host-native semantic views: real new software around retained engines

Where Eversion has enough semantic access, build a new native view specialized to the workflow: a multi-source comparison, cue inspector, claim-evidence editor or conflict resolver. The source app still performs its actual domain operation. This is purposeful re-presentation of selected capabilities, not an attempt to recreate the whole application.

A native inspector can be more composable than the original control. For example, expose a verified DAW parameter with physical units, bounded range, automation state and source identity; create an aggregate control over several parameters using an explicit mapping. If the only evidence is a cropped knob, retain it as an opaque live surface. The architecture never silently promotes visual resemblance into a typed parameter.

## 6. Rendering and interaction are one contract

### 6.1 Surfaces have lifetimes

Define a transport for surface leases, frame sequence, source generation, geometry epoch, device scale, orientation, alpha, color space, damage and acquire/release synchronization. The compositor does not retain source buffers beyond their documented lifetime. Stock CEF's accelerated rendering callback explicitly requires copying a pooled macOS IOSurface into client-owned storage before the callback ends; the honest initial objective is GPU-to-GPU copying without CPU readback, not universal zero-copy. [CEF render handler](https://github.com/chromiumembedded/cef/blob/master/include/cef_render_handler.h).

Enqueueing an asynchronous Metal copy is insufficient if it still uses the CEF source after callback return; complete source-resource use within the allowed lifetime unless an explicitly supported synchronization contract extends it.

An owned producer can support a stronger resource lease with release after Metal work completes. Chromium's transferable-resource definitions show why resource release and GPU completion need explicit synchronization. Apple's IOSurface Mach-port transport supplies a native sharing building block, not a UI or permission protocol. [Chromium transferable resources](https://raw.githubusercontent.com/chromium/chromium/main/components/viz/common/resources/transferable_resource.h), [IOSurface Mach ports](https://developer.apple.com/documentation/iosurface/iosurfacecreatemachport(_:)).

Bound frame queues; coalesce intermediate visual updates and retain the newest valid frame. Do not coalesce effect receipts, object deletion or workflow transitions. Renderer death and GPU reset revoke surface generations. A retained last frame is a labeled snapshot, not proof that its controls are still actionable.

Avoid rendering every hidden panel at maximum refresh. Maintain subscriptions and workflow state independently from visual demand. The scheduler can reduce compositor frame demand for inactive views, but source-side background throttling must remain observable. This is separate from a page's visibility/focus lifecycle: define aggregate visibility when one document has several projections, and test visibilitychange, intersection-dependent rendering, viewport virtualization and focus-dependent updates. Keeping a projected source viewport active may be necessary for compatibility and has a real resource cost. A native source may stop rendering; no compositor can manufacture current content it never produced.

### 6.2 Act against the frame and identity actually presented

A pointer event names the projection and geometry/frame generation the person saw. An owned producer retains a frame-associated hit-test/object map, binds the target incarnation, and validates its continuity at dispatch. Same-position replacement is invalidation even when geometry is unchanged. If that correspondence is unavailable, decline the event or require an explicit weaker interaction contract. Owned browser/runtime paths can perform a stronger coordinated check than an external native crop. External checks reduce but cannot eliminate a race between observing geometry and posting an event; never advertise them as atomic.

Paint selection and hit-test eligibility must agree. If a projection omits an originally occluding sibling, it must either preserve original occlusion or declare a new interactive presentation whose eligibility follows its selected paint/interaction envelope. Dispatch can retain the original DOM/framework propagation path, but it must not send a visible control's gesture to an invisible excluded occluder. This change in presentation semantics is explicit and requires conformance testing.

Treat a whole gesture as a session: pointer capture, drag negotiation, scrolling phase, modifier state and cancellation. The surface owns an **interaction envelope** containing popups, menus, sheets, candidate UI and related dialogs. If the backend cannot export it, expand or reveal the original interaction rather than guessing which popup belongs in the crop.

Browser security UI is a separate authority channel. Permission prompts and authentication chrome must identify the requesting origin; projected source menus do not substitute for them. Preserve browser user-activation requirements and distinguish a contemporaneous human gesture from an agent operation. Clipboard, fullscreen, popup and permission-sensitive actions may require an explicit trusted browser handoff. Owning the engine is not a reason to silently disable these checks. [WHATWG user activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation).

Text input needs a complete contract for marked text, selection, replacement ranges, commit/cancel and caret geometry. A host NSTextInputClient is appropriate only where the adapter can preserve those semantics. Otherwise the original source owns the text session. Sending committed Unicode is useful but not equivalent to forwarding composition behavior. Open/save panels also carry authority: a host file picker does not automatically grant a sandboxed target access to the selected file. [Apple text architecture](https://developer.apple.com/library/archive/documentation/TextFonts/Conceptual/CocoaTextArchitecture/TextEditing/TextEditing.html), [NSOpenPanel](https://developer.apple.com/documentation/appkit/nsopenpanel).

The host supplies an accessible tree for its own controls and projected semantics, with correct transformed geometry and source action routing. A texture does not inherit a source's accessibility. An opaque surface remains honestly opaque and can offer a route to the original app's accessible UI; fabricated labels are not equivalent accessibility support. [NSAccessibilityElement](https://developer.apple.com/documentation/appkit/nsaccessibilityelement-swift.class).

## 7. The workflow language: dataflow, cases and effects

### 7.1 One owned IR, several cooperating semantics

Use a typed, serializable intermediate representation that a visual editor, text editor and agent can all edit. It needs:

- source declarations and required capability contracts;
- source-specific and Eversion-owned entity schemas;
- explicit relationship predicates and provenance;
- pure incremental queries and transformations;
- hierarchical statecharts instantiated per case/entity;
- durable effects with authority and confirmation requirements;
- presentation rules over case state and entity selection;
- membership, versioning, migration and resource policies.

Pure dataflow handles “show all claims affected by this source change.” A case machine handles “this claim was reviewed, then changed, so request re-verification.” An effect handles “apply this approved edit to this current source revision.” These must not collapse into a graph in which any changing cell can emit an arbitrary external write.

Hierarchical and parallel statecharts supply established semantics for nested lifecycles and concurrently active concerns. SCXML is a useful reference for transition priority and run-to-completion, not a requirement to adopt XML or an existing orchestration product. Eversion should specify its own small executable subset and give each macrostep a bounded computation budget so a self-triggering rule cannot monopolize the UI. [W3C SCXML](https://www.w3.org/TR/scxml/).

Use a canonical local event/commit sequence and a snapshot-consistent graph frontier for each macrostep, including shared derivations, child creation and cohort membership. Independent partitions may evaluate concurrently only where ordering is immaterial or a transactional commit check preserves the recorded order. This local ordering does not claim a global order in external sources. Replay consumes recorded ordering and membership rather than querying the current graph. Exhausting a macrostep budget suspends it without committing a partially evaluated transition/effect set.

Each case processes a logged external event, updates pure derivations, chooses transitions and atomically persists local state plus new effect intents. Only then may the effect broker dispatch them. Outcomes return as events. Source reads, timer firings, agent responses and nondeterministic choices enter the recorded history. Replaying history reconstructs local decisions without repeating external effects.

### 7.2 Dynamic topology is part of the program

A relation can create or retire a child case and its views. When a new research claim appears, instantiate its verification machine; when a cue splits into two, instantiate descendants and preserve lineage. A parent may wait for a defined cohort, not just count whatever happens to be visible in a list.

Membership policies include:

- a snapshot of members at a named revision;
- all members observed before an explicit closing event;
- a live membership set that never implies completion merely because temporarily empty.

Absence is meaningful only with coverage evidence. A virtualized row missing from the current viewport is not deleted. “All reviewers approved” must evaluate to unknown if the reviewer set cannot currently be enumerated completely. An empty response after disconnection must never complete the case.

A live rule edit is a new program version. Pin active cases to their version until a migration defines what happens to their state, obligations and pending effects. Migration can apply only to future cases, pending cases, or a specified set of current cases. Never replay past entry actions merely because a new graph was installed. In-flight effects remain bound to their issuing version; migration must either wait for settlement or explicitly retain the old operation/receipt lineage and map its eventual outcome into the new case without reissuing it.

Adding a view is cheap; changing a behavior can change obligations. The user-facing editor should show concrete consequences: “these three claims need a second source; this completed publication becomes a follow-up case.” This makes changing behavior as tangible as dragging a panel.

### 7.3 Example IR sketch

This is illustrative language design, not executable application code:

```
case VerifyClaim(claim: Claim, policy: ReviewPolicy@version) {
  evidence = observe claim.supportingExcerpts
  ready = complete(evidence, scope = claim.requiredEvidence)
       && satisfies(evidence, policy.requirements)
       && every(evidence, isCurrent && supports(claim.revision))

  state Researching {
    on EvidenceChanged when ready -> Reviewing
  }
  state Reviewing {
    on Approved(scope = claim.revision + evidence.revisions,
                attempt = currentReviewAttempt) when ready -> Accepted
    on ClaimChanged -> Researching
    on DependencyChanged -> Researching
  }
  state Accepted {
    on DependencyChanged -> NeedsReverification
  }

  view = original(claim.editorRange)
       + evidenceViews(evidence)
       + actionsAllowed(currentState)
}

case Publish(article: Article) {
  cohort = snapshot(article.claims, on = StartPublication)
  require cohort.complete
       && every(cohort, acceptedFor(article.frozenRevision))
  staged = await effect StageDraft(article.frozenRevision)
    target destination.documentIdentity
    confirm receipt(DocumentId, Revision)
  await effect Release(staged.documentId, expected = staged.revision)
    grant publicationPolicy
    confirm receipt(ReleaseId)
    reconcile destination.releaseIdentity
}
```

The compiler rejects the second effect if no implementation can provide its required target identity, grant or reconciliation semantics. The user can consciously choose a manual source step with a different completion contract, but the compiler cannot silently weaken it.

### 7.4 Durable effects and unknown outcomes

An effect binds an immutable target identity/incarnation, arguments, input revision set, program/adapter versions, authorization scope, preconditions, idempotency behavior, required success evidence, deadline and any compensation. Repairing a future locator does not retarget an already issued effect. Immediately before dispatch, revalidate the current grant, required approvals, target binding and relevant preconditions. Revocation stops queued effects; already-dispatched ones still require outcome reconciliation.

Every asynchronous request/result carries a request ID, attempt generation, case/program version and dependency frontier. A delayed agent answer, export or read remains evidence for its original request; if superseded, it cannot silently advance current state or authorize a new effect. Old effect receipts remain valid historical evidence for the original operation and must still be reconciled even when a case has migrated.

Track at least `prepared`, `dispatched`, `locallyAccepted`, `confirmed`, `rejected`, `outcomeUnknown` and `reconciled`. A particular capability may not distinguish every state; the evidence field says what is known. Only declared confirmation evidence advances dependent work. A successful AX call may establish dispatch rather than completion.

Use source-supported idempotency keys when available. Where a source supports authoritative status lookup, reconcile before retrying. Where it exposes neither, an interrupted irreversible effect remains unknown and cannot be blindly repeated. A confirmed staging receipt remains recorded if a later release fails or becomes unknown; recovery resumes only the unresolved step. A local journal cannot manufacture exactly-once remote execution. Cancellation prevents future dispatch; compensation is a new operation with its own possible failure.

Conditional writes deserve a stronger type than preflight checking. Google Docs, for example, documents revision controls and atomic application of one request batch. Those guarantees are local to the Docs operation, not a transaction spanning an editor, note app and publication service. [Docs batchUpdate](https://developers.google.com/workspace/docs/api/reference/rest/v1/documents/batchUpdate).

Approvals bind to the content, destination and dependencies actually approved, or to an explicit standing policy that permits a class of changes. Changed dependencies invalidate only the affected approvals. Authorization need not mean constant prompts: a user may grant bounded unattended behavior in advance. The runtime enforces that grant's scope and records its use.

Undo should distinguish local program edits, local proposal edits, source-supported undo, and compensating external actions. It cannot truthfully offer a single desktop-wide rewind. Forking a composition forks its local program/proposals/history reference; live source data remains shared unless the source supplies a real branch or clone.

### 7.5 Continuous control is a separate execution path

Reactive creative work includes scrubbing, transport following, parameter mapping and live preview. Model these as explicit controllers with clocks, sampling rate, units, bounds, smoothing/deadband, feedback policy, ownership and loss-of-observation behavior. A transport follower is not a retryable message-send action.

The host can compile a verified mapping into a source plugin or dedicated local controller. Agents propose controllers and inspect traces; they do not sit in an audio callback or decide each animation frame. Clock transforms between wall time, video frames, musical beats and audio samples require a declared reference and drift treatment. When the source supplies no sample-accurate scheduling, the interface must not imply it.

For example, Max for Live's observer has specific constraints: not all properties are observable, its notifications cannot directly modify the Live set, and its API messages run deferred on the main thread. An Eversion controller must honor that execution model rather than treating every observed change as permission for immediate reentrant writes. [live.observer](https://docs.cycling74.com/reference/live.observer).

## 8. Agents create compositions and learn adapters

An agent should be a participant in software construction, not merely a user impersonator pressing buttons. Give it an **Eversion workbench** with a capability catalog, permitted source observations, a recorder, a trace simulator, a compiler and an adapter test harness.

### 8.1 From an example to a running program

1. The person names a goal or demonstrates a relationship: “when this cue changes, gather affected reviews and prepare new exports.” The agent records the intended entities, temporal behavior and examples, not just the click sequence.
2. Discovery interrogates permitted channels of the actual tools. It identifies runtime families, available operations, observable revisions and missing evidence. It offers the useful contract currently achievable and names what deeper access would add.
3. The agent proposes domain entities, mappings, case machines, controller rules and views in the owned IR. The interface shows ordinary examples: a new cue, a late review, a source relaunch, a failed export.
4. Static checks reject unit mismatches, unspecified authority, unsafe feedback, missing unknown handling, ambiguous membership and unsupported effect guarantees. Recorded or synthetic traces exercise branches and failures.
5. Deploy a versioned composition under already granted scopes. New permissions or new consequential behavior require a concrete scope change, not a hidden broadening inside generated code.
6. During use, the agent explains blocked cases, proposes repairs and refactors repeated patterns into reusable parts. Each deployed repair is a program change with compatibility evidence.

A workflow's central behavior becomes inspectable and editable by both person and agent. The person need not write a statechart: demonstration, natural language and concrete examples are alternative authoring interfaces to the same program.

### 8.2 Discovery should be intervention-based when authorized

Names and tree structure suggest possible semantics; they do not establish them. Build a capability-learning loop that compares before/after observations around an authorized operation in a test document or disposable source instance. Correlate model changes, UI updates, local acknowledgments and eventual remote state. Generate alternative explanations and test counterexamples.

A learned `setFilter` mapping must distinguish filtering a view from changing server-side records. A learned “save” must distinguish local dirty-state clearing from confirmed persistence. A discovered command closure should be invoked only on its expected thread and lifecycle, and only after its side-effect envelope is understood sufficiently for its scope.

Generalize the mechanism at the runtime/framework level, but keep domain mappings explicit. A rich-text framework adapter can expose logical ranges and edit batches across applications; deciding that a particular range is “the legal approval paragraph” remains a workflow binding. A Qt adapter may find observable models; interpreting a model row as a musical cue requires more evidence.

**No runtime access is presumed just because an agent can write a probe.** Probe admission is a separate capability. Tests cannot certify hidden server semantics they cannot observe. A successful demonstration produces a candidate adapter with bounded applicability, not universal approval.

### 8.3 Agents have no ambient authority

The agent sees only scoped observations and invokes typed broker operations. Source text, web pages, document comments and plugin metadata are untrusted content. They cannot modify authorization or instruct the agent to export unrelated data. Generated UI also has no raw access to privileged IPC merely because it displays a source surface.

Separate code proposed by an agent, code admitted by the validator, and code executing with source permissions. Keep secrets in the owning session or credential store; adapters use handles rather than embedding tokens in composition files. Exportable compositions contain schemas, rules, symbolic bindings and synthetic fixtures. Actual observations, account identifiers and content remain local unless the user chooses to share them.

Repair can replace a locator after confirming identity; it cannot reinterpret a pending effect or expand a grant. If compatibility is lost, revoke the affected capability and leave unrelated case branches running. The explanation should identify the lost ability—such as “cannot verify current cue identity”—rather than show a generic automation error.

## 9. Three workflows that evolve while running

These are proposed acceptance scenarios, not claims that every named tool currently exposes the required contract. Their adapter requirements are part of the tests.

### 9.1 An argument workspace becomes two publications

A researcher starts with a browser, PDFs, a notebook, a native figure editor and an unsaved text editor. Eversion creates `Claim`, `Excerpt`, `AnalysisRun`, `Figure`, `Review` and `Publication` entities around the real tools. Selecting a passage creates a claim at the live buffer's current revision; selecting evidence in a PDF/browser attaches a versioned excerpt. VS Code's extension API provides one concrete path to document changes, dirty state and versions. A disk watcher alone is not an equivalent live-buffer adapter. [VS Code API](https://code.visualstudio.com/api/references/vscode-api#TextDocument).

The workspace is already behavioral: selecting a claim brings related excerpts, notebook results and its original editor surface into view. Adding a quantitative claim instantiates a verification case. Running an analysis creates a revision-bound output; the figure view carries that provenance. The figure editor retains its specialist tools.

The researcher splits a claim into two. Eversion records lineage, creates two child cases and asks the agent to propose which evidence supports each. It does not transfer approval merely because much of the wording survives. Meanwhile, a changed source invalidates only dependent excerpts and figures. Unaffected writing continues.

A reviewer adds a requirement that quantitative claims need two independent sources. The agent proposes a rule migration and shows the affected work. Three claims return to verification; an already released article produces a correction/follow-up case. The historical publication remains unchanged.

The researcher forks a short and technical version. They share source entities and some evidence while retaining different argument structures, proposal text, review scopes and publication cases. Selecting a paragraph in one does not overwrite the other's selection. If the source editor cannot allocate independent views, the host uses a semantic comparison view or an explicit second document rather than duplicating one scroll crop.

At publication, freeze the actual buffer/artifact revision, generate outputs, verify them and stage the destination. If the destination accepts release but the reply is lost, Eversion resumes reconciliation after restart. It does not repeat publication. A still-open PDF snapshot cannot resolve an unknown remote outcome.

The novelty is the evolving network of claims, evidence, computations, reviews and source capabilities. The same composition changes its own obligations and interface as the work grows.

### 9.2 A film-scoring workspace reacts to picture changes

A composer combines a video timeline, DAW tracks, original plugin controls, a web review service and a cue sheet. Eversion introduces a `Cue` identity linked to a video range, DAW clip/automation region, export artifacts and review threads. It owns the correspondence; each source owns its actual media/project data.

A selected cue opens the actual plugin and timeline interactions needed for that cue. A typed inspector exposes verified parameters with units and automation state. A rehearsal controller follows transport through a clock mapping. Opaque controls remain original surfaces; the agent cannot infer hidden preset state from a knob's appearance.

An updated picture cut moves several boundaries. Eversion marks the old cue-to-shot mapping stale, proposes a correspondence, creates a repair case for uncertain matches, and recalculates only derived timing that has a declared transform. It does not overwrite artistic edits in the DAW. The composer accepts some proposed shifts and manually resolves a split cue, yielding two child cue cases.

While the composer auditions one branch, an isolated or source-supported second instance renders alternatives. If the source cannot branch its project, comparison uses immutable renders plus local proposals; it does not advertise two independently editable copies of one live state. The agent can generate a new controller rule—for example, a bounded mapping between a gesture and several verified parameters—and test it against recorded control traces before deployment.

Review arrives during rendering. Feedback on an obsolete render attaches to its render revision; Eversion suggests whether it still applies. A late change invalidates export approval only for affected cues. Export scheduling and review logic run in the case runtime; sample-accurate audio remains in an appropriate source/plugin path. If a modal plugin editor claims the source seat, other semantic tasks continue while conflicting gestures wait.

This scenario falsifies a system that only swaps screen arrangements: it requires temporal identity, controller behavior, concurrent alternatives, dependency invalidation and durable long-running cases.

### 9.3 An exception desk assembles itself around new cases

A user works with an existing supplier portal, a local spreadsheet and an internal issue tracker. Eversion begins with typed host views where data is available and attached original surfaces where it is not. An agent proposes a mapping between supplier IDs, order rows and issue references; ambiguous matches are shown as unresolved relationships.

A delayed shipment creates an exception case. The case allocates a comparison view, gathers related contract evidence and opens only the source capabilities required for its current state. Two suppliers with identical names remain distinct by account-scoped IDs. If the portal only exposes a virtualized list, the workflow cannot infer that all open orders have been examined.

The user adds a rule during the day: high-value cases need an extra review, while routine cases may prepare a standard response under a standing policy. The agent simulates the rule, migrates eligible active cases and adds a new decision stage. Existing drafts are proposals tied to evidence versions; source changes invalidate their affected claims.

An account switch invalidates portal grants and pauses only dependent cases. After reauthentication, source identity is re-established before rebinding. A response operation whose remote outcome is unknown stays unresolved until reconciliation or an explicit human decision. Cases can resume tomorrow with the original decision history even if the sources all opened in new windows.

The result is a changing application organized around work instances. Its components, available actions and topology arise from case state and relationships, rather than a predetermined set of screens.

## 10. Alternative architectures and why I choose this one

| Alternative | Real advantage | Cost or failure mode | Decision |
|---|---|---|---|
| Capture-and-forward desktop shell | Broad initial reach; original visual behavior remains present | Shared source input/state; little domain meaning; fragile dynamic behavior and recovery | Keep as an attachment backend and discovery surface, not the product's semantic foundation |
| API-first integration hub | Stronger schemas and outcome evidence where vendors cooperate | Misses unsaved state and specialist interactions; predetermined integrations dominate | Use available APIs inside the common contract, without making their existence a prerequisite |
| Universal runtime injection | Potentially deep client access, including closed-source behavior | Admission, ABI, lifecycle and application assumptions vary; remote authority remains outside | Invest in framework families and narrow admitted probes; reject universality as an unsupported promise |
| Full desktop/VM per app | Strong input and lifecycle isolation | New execution identity, account/session work, hardware/media and interaction costs | Allocate selectively by interaction domain; measure before using for continuous direct manipulation |
| Own browser only | Greatest coherent control of web painting, routing and instrumentation | Authentication acceptance and native applications remain outside; existing sessions cannot be assumed portable | Strategic engine investment plus permanent existing-session attachment |
| Reverse-proxy rewriting | Can alter pages and integration boundaries extensively | Changes origin/session semantics and expands traffic trust; sophisticated apps require extensive virtualization | Specialist backend only when its actual origin/auth/security contract is acceptable |
| DOM transplantation/mirroring | Easy-looking reusable fragments and flexible layout | Node identity is not execution closure; event, framework, canvas and document dependencies remain | Do not base general behavior on transplantation; use verified adaptations or engine projections |
| New universal application substrate | Clean types, state and composition when all tools participate | Requires replacing/adapting the user's ecology before value appears | Use a small owned kernel around real engines, with native host views only where useful |
| Agent operates GUI on demand | Very broad procedural flexibility | Weak persistence and repeatability if reasoning remains the execution substrate | Agents compile and repair programs; GUI operation remains one explicit capability implementation |
| Pure dataflow graph or pure statechart | Each has a coherent model | Dataflow alone mishandles effects; monolithic statecharts explode with dynamic entities | Combine pure derivations, per-case machines, effect ledger and separate continuous controllers |

The strongest rival is an automation-centered product with excellent native controls and no deep projection engine. It may capture much of the value sooner. I still recommend investing in the owned projection/runtime path because retained specialist interaction is central to the stated intent, and because browser/native framework control can enable forms of composition a generic automation layer cannot. That investment should survive only if the fidelity and generalization experiments below succeed.

“More own control” should mean ownership of the invariants that define the product: identities, contracts, cases, resource routing and presentation. Reimplementing TLS, all of AppKit, or a browser's full web platform would add ownership without necessarily increasing useful control. Maintaining a focused engine fork or target-side protocol is different: it directly changes what compositions can express.

## 11. Experiments designed to overturn the recommendation

None of these experiments was executed for this document. The targets below are proposed acceptance criteria and decision rules, not measured capabilities. Use synthetic documents and separately authorized test accounts/targets; this architecture task does not authorize installations or changes to the user's system.

| Experiment | Method and observable result | Falsification / architectural response |
|---|---|---|
| **Native input contract** | Across AppKit, Qt, SwiftUI and custom-rendered fixtures, exercise IME, drag, scroll, shortcuts, menus, sheets and file panels through two portals while the human types in a host field. Vary focus and source window lifecycle. | Any silent wrong-target mutation disqualifies that backend from unattended input at that scope. Broaden leases, hand off full interactions or require a deeper backend. |
| **Framework reuse** | Produce probes for three materially different apps in each chosen runtime family; test a supported version change. Record how much behavioral code is framework-generic versus app-specific. | If most successful capabilities depend on bespoke object/command knowledge, retain generic probes as discovery tools and budget semantic integration per app; stop claiming framework-wide extraction. |
| **Owned browser projection** | Controlled corpus: nested scroll, portal menus, shadow DOM, cross-origin frames, canvas, sticky layout, transforms, blend modes, masks, filters, omitted occluding siblings, visibility/intersection-driven updates, IME and accessibility. Compare source event/state traces and visual output to ordinary rendering. | If projection requires changing ordinary source semantics or loses essential interaction dependencies, narrow projections to supported classes or preserve complete page contexts. A crop demo does not pass. |
| **Same-position replacement** | Present a frame, replace its button/object without changing geometry, then dispatch an event associated with the old frame. Interleave renderer/main/compositor updates. | Dispatch to the replacement is a failure. Require frame-associated object maps and continuity checks; weak attachment paths cannot claim this guarantee. |
| **Authentication acceptance** | Test ordinary login, federated OAuth, passkeys, enterprise restrictions, account changes, session expiry and recovery in both managed and attached modes. Record actual permitted paths. | If important tools reject the managed environment, browser attachment remains the dominant backend for those tools. Do not patch out policy or claim external OAuth solves session transplantation. |
| **Identity and membership** | Rename, reorder, duplicate, virtualize, delete/recreate and relaunch bound objects; include observation gaps and unobservable account state. Interrupt an enumeration before a completion gate. | Any silent wrong-entity action or treating incomplete membership as empty fails. Unknown continuity must suspend affected capabilities; inability to establish continuity limits supported automation. |
| **Effect crash matrix** | In fixtures, terminate broker/kernel/source before send, after send, after commit, before receipt persistence and during reconciliation. Include non-idempotent operations. | Unsupported success, blind duplicate execution, retargeting after repair or false global rollback fails the kernel design. Preserve unknown outcomes and inspect which source contracts permit recovery. |
| **Version and migration** | Replay cases with concurrent unsaved editor changes, mismatched preview builds, new cohort members, late reviews and a live rule edit. Maintain an independent expected-obligations oracle. | Lost obligations, stale approvals, repeated entry effects or past effects executed during replay fail. Repair IR semantics before expanding adapters. |
| **Resource and deadlock** | Construct cyclic multi-source workflows; interrupt gestures and pending resource-set acquisition; slow an agent and hold a modal dialog. | An indefinite lease or wrong-owner event fails. Release ordinary leases before waits, acquire resource sets atomically/in order, and bound queues. |
| **GPU and lifecycle** | Resize and change display scale continuously; delay GPU completion; kill renderers; reset GPU processes; minimize/hide native sources. Measure retained memory and source/frame associations. | Invalid buffer access, unbounded retention or actionable unlabeled stale frames fails the surface protocol. A GPU copy is acceptable if measured performance supports it. |
| **Isolation payoff** | Run the scoring and argument scenarios in attached apps and isolated domains. Measure input independence, source feature loss, recovery, p95 interaction latency and resource cost. | If isolation sacrifices required media/hardware fidelity or responsiveness, limit it to asynchronous operations and retain local original interactions. |
| **Agent construction and containment** | Ask agents to construct the same workflow across unrelated tools, then repair version changes. Include malicious source instructions, false labels, revoked grants and ambiguous entity matches. | Silent grant expansion, misplaced effects or opaque repairs fail. Measure retained IR versus rewritten domain logic; low reuse falsifies the claimed generalization boundary. |
| **Longitudinal product test** | Run the evolving research and scoring workflows for two weeks, including restarts, source changes and interruptions. Measure task completion, repair time, unresolved uncertainty and users' ability to explain active behavior. | If maintaining Eversion costs more effort than the original workflow or users cannot predict consequential behavior, a polished compositor has not validated the product. Simplify semantics or target more observable domains. |

Performance measurements must distinguish source rendering, transfer, composition, input dispatch, source processing and semantic propagation. As an initial engineering target, test a native host with eight source contexts and twenty-four visible/latent projections, aiming for compositor overhead below one 60 Hz frame at p95 under a declared workload. This is a budget for Eversion's overhead, not a promise about remote-service latency or hard real-time audio. Measure GPU memory, copies, energy and pressure as well as frame rate. Dedupe shared source capture and avoid multiplying full-window buffers per crop.

A conformance matrix should report which capabilities pass each test on which source/OS/build. A global percentage such as “supports 95% of apps” would hide the difference between read-only display, safe typed edits and fully independent interaction.

## 12. What to build first, if the architecture is approved

The first implementation should be a **vertical proof of dynamic composition**. It should include the owned IR/kernel and effect ledger from the beginning, plus a native source, an attached browser source and an editor with live-buffer semantics. Implement one workflow that creates cases, changes relationships, survives a source restart, invalidates an approval on changed inputs and recovers from an unknown operation outcome. A large catalog of crop adapters would not test the thesis.

In parallel research tracks after authorization, prototype the owned browser projection API and one framework-native deep adapter. Use their results to decide where control actually generalizes. A third track can test isolated execution only where the example needs independent interactive state. These experiments should use the same capability contract so unsuccessful backends can be removed without rewriting the workflow model.

The order is driven by uncertainty:

1. Establish identity, effect and migration correctness on controlled fixtures.
2. Demonstrate useful dynamic workflow construction around real tools using the strongest available channels.
3. Establish projection/input fidelity and authentication coverage.
4. Measure framework-level reuse and the agent's adapter-development contribution.
5. Only then expand compatibility and invest in polished authoring surfaces.

This does not mean minimizing ambition. The desired endpoint is a user-programmable environment in which a new relationship can create a new tool, a changed source can change obligations, an agent can author a verified behavior, and original specialist software remains present wherever its interaction is valuable. The disciplined contract boundary makes that ambition survivable when sources disagree, disappear or refuse access.

### Decisions I would carry into joint review

- Eversion owns an executable composition model; a layout is one projection of that model.
- Existing apps retain domain authority; Eversion owns explicit cross-app relations, proposals and workflow state.
- Attached and managed execution are permanent, complementary modes.
- Build a native compositor and pursue an owned browser projection layer; retain complete source execution closures initially.
- Treat framework probes, semantic views and isolated instances as distinct mechanisms with different admission and independence guarantees.
- Make identity continuity, partial observation, input resources, effect evidence and migration semantics part of the public component contract.
- Agents construct, test and repair versioned programs and adapters; they do not replace the deterministic or real-time runtime.
- Do not claim universal independent views, automatic semantic inference, universal undo, universal injection or exactly-once effects across uncooperative sources.

The most consequential open question is whether useful identity, behavioral and outcome contracts can be acquired across enough valuable tools with tolerable repair work. The architecture is designed to expose that question early, capability by capability, rather than hide it behind a convincing screen composition.

## Supporting independent research

- [Native control](native-control.md): native actors, framework probes, AppKit/Qt, target admission, session isolation and five experiments.
- [Web runtime](web-runtime.md): execution-closure retention, engine projections, GPU resource lifetimes, authenticated-session modes and falsification.
- [Workflow semantics](workflow-semantics.md): identities, coverage, statecharts, effects, migration, agent compilation and an evolving writing workflow.
- [Input verification](input-verification.md): exact approved-file hashes and evidence boundary.

All external links in this draft and the supporting memos point to primary documentation, primary project sources or author research. Where a mechanism is inferred or proposed, that status is stated. No app implementation, installation, target instrumentation, system security change, Git mutation or publication was performed in this stage.
