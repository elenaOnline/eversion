# Native control: owning the composition boundary

Astra independent research memo, 2026-10-07. Read intent, brief, native/modding/critique findings; principal verified packet hashes. No recommendations or other architect work read. No implementation, target inspection, or system changes performed. Documented mechanisms and proposed engineering are separated below.

## Recommendation

Build a native composition server that owns presentation, interaction arbitration, workflow state and capability contracts. Model each source as an **actor with a state boundary**, rather than as an arbitrarily detachable visual subtree. Give each actor three independently negotiated interfaces: observable facts, executable commands, and live surfaces. A source can supply any subset.

This is deliberately more ambitious than a capture shell: eversion owns the workflow’s real state machine and native controls, while adapters expose actual source behavior. It can replace a source presentation where semantics are known, retain an interactive surface where behavior is opaque, and move an entire source instance into a controlled environment when independent interaction is essential. These are distinct mechanisms, not a universal extraction ladder.

## 1. The owned boundary

Proposed protocol, not an existing Apple API:

- **Identity:** actor instance, document/account namespace, opaque object ID, generation, adapter/build fingerprint. A pointer, AX reference or coordinate never becomes durable business identity by itself.
- **Observation:** typed value, source revision, observed time, provenance, completeness and invalidation. Partial observations stay partial.
- **Commands:** input schema, precondition, expected state transition, acknowledgment level, cancellation boundary and reconciliation query. “Event dispatched” is distinct from “local model changed” and “remote commit confirmed.”
- **Surfaces:** stream identity, frame sequence, geometry epoch, content bounds, color/scale metadata, stale/hidden status and related popup surfaces.
- **Interaction:** lease owner, affected state scope, active selection, text composition, pointer capture, modal descendants and release conditions.

Use an owned AppKit/Metal host; keep adapters in separate helper processes where possible. A native probe necessarily executes some code inside its target, so a helper boundary cannot protect that target from probe bugs. Treat probe versions as dependencies with explicit compatibility and rollback.

Display surfaces as capabilities with lifetime, not screenshots with implied freshness. Apple documents secure task-to-task IOSurface transfer by Mach port; this supports a custom producer/consumer surface transport without making buffers globally discoverable. It does not transport NSView objects, event semantics or permission. [IOSurfaceCreateMachPort](https://developer.apple.com/documentation/iosurface/iosurfacecreatemachport%28_%3A%29?changes=__2)

## 2. Three native backends

| Backend | Real ownership obtained | What remains shared |
|---|---|---|
| External actor: AX, scripting, window capture | Workflow, host controls, supported source commands | Existing process, document state, modal state and desktop input |
| Framework actor: permitted in-process probe | Object observation, command interception, potentially framework rendering and explicit event dispatch | Original process/runtime, object lifetimes and unspecified app-global assumptions |
| Isolated actor: separately launched app/session or macOS guest | Instance lifecycle; a guest adds independent desktop focus/session | Remote service/account state unless separately partitioned |

For an external actor, collect capture, AX and scripting as complementary channels. An AX button action is preferable to a coordinate click only if it implements the intended command; neither grants durable transaction semantics. Prefer native eversion controls backed by verified operations for frequently composed behavior. Keep an original-app surface for behavior whose semantics are not yet reconstructed.

For a framework actor, discover runtime family and compatibility before installing a probe. Frida documents Objective-C object/method access, implementation interception, RPC and scheduling on the main queue. These are enough to build a research probe for an admitted target; they do not prove a general permission to enter arbitrary production apps. GammaRay’s matching-probe selection and refusal of incompatible Qt versions is a useful operational precedent. [Frida API](https://frida.re/docs/javascript-api/), [GammaRay launcher](https://docs.kdab.com/gammaray-manual/latest/gammaray-launcher-gui.html)

The probe should first export application behavior: selected objects, action dispatch, model notifications and narrow commands. Only then attempt graphical export. This is how “rename selected asset” can remain meaningful while its old toolbar disappears or a workflow creates a richer bulk-rename editor.

**AppKit hypothesis:** keep the original view in its original hierarchy and export its appearance plus semantic operations first. If necessary, create a target-owned hosting window or a second presentation bound to the same model. Do not promise that detaching a view preserves constraints, responder behavior, delegate assumptions, undo or document associations. Apple's bitmap caching API draws a view and descendants, but that is evidence for a snapshot path, not a universal live offscreen renderer for Metal/video/system effects. [NSView bitmap caching](https://developer.apple.com/documentation/appkit/nsview/cachedisplay%28in%3Ato%3A%29?language=objc)

**Qt path:** QQuickRenderControl explicitly supports application-controlled offscreen rendering into textures, including integration with Metal. It preserves a QQuickWindow for scene management/event delivery; events can be sent to it and focus set explicitly. Popup placement still requires a real-window/offset mapping. This is a concrete substrate for a framework-specific export adapter. Retrofitting an existing app’s scene, recreating its bindings and acquiring a compatible in-process position remain engineering hypotheses. It proves neither arbitrary QWidget extraction nor universal Qt app compatibility. [Qt render control](https://doc.qt.io/qt-6/qquickrendercontrol.html)

For SwiftUI/custom GPU apps, begin with external semantics or app-specific runtime/model adapters. Do not infer that discovering an NSHostingView yields a stable public SwiftUI state graph.

## 3. Own input as a resource, not a coordinate transform

AppKit routes keyboard input through key equivalents, navigation and first responder; pointer gestures and actions take different paths. Reparenting also changes responder structure. PID-targeted delivery alone cannot establish correct component behavior. [Apple event architecture](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/EventOverview/EventArchitecture/EventArchitecture.html)

Proposed scheduler: an actor declares an **interference scope**—element, document, process or whole desktop session. Semantic operations proven independent can run concurrently. Pointer/keyboard sessions acquire the appropriate exclusive lease until gesture, composition or modal workflow ends. Multiple crops sharing one source document retain their shared scroll, selection and undo state; the UI should expose that coupling. Never advertise them as independent copies.

For text, eversion can own an NSTextInputClient only when an adapter supports marked-text ranges, selection, replacement and geometry with adequate synchronization. Apple’s text architecture explicitly requires cooperation with input context and candidate positioning. Otherwise route a complete text session to the original source, with visible focus ownership. Splitting IME handling between host and source without a composition contract is an untested design, not an implementation detail. [Apple custom text views](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/TextEditing/Tasks/TextViewTask.html)

Represent menus, tooltips, sheets, file panels and drag previews as temporary descendants of an interaction, not accidental windows to crop later. A capture-only backend may need to reveal the real app for such interactions. A probe can potentially export them. A controlled guest can contain them.

## 4. The protected-runtime boundary

An eversion entitlement does not authorize loading a library inside a different vendor’s process. Apple’s library validation normally restricts loaded code to platform code or the same signing team; opting out is a property of the target. Squish’s own macOS documentation confirms that its hooking requires target DYLD allowance and appropriate library-validation treatment. [Apple DTS explanation](https://developer.apple.com/forums/thread/126895), [Squish hardened-runtime requirements](https://qatools.knowledgebase.qt.io/squish/mac/troubleshoot/hardened-runtime/)

Admit probes through a supported plugin mechanism, developer cooperation, or an explicitly managed compatible copy where permitted. Re-signing is a new execution identity: assess keychain access, app groups, updates, licensing and server trust experimentally. It is not equivalent to keeping the original signed application. Do not make weakening the user’s normal system security a baseline product dependency.

An owned VM does not automatically remove the guest’s target-process protections. Its architectural benefit is control over lifecycle and a separate desktop, not a magical injection privilege.

Apple supports macOS guests on Apple silicon through Virtualization. Current documentation also permits iCloud in qualifying macOS 15+ VMs; identity is tied to host Secure Enclave information, and moving/cloning can require reauthentication. Therefore neither “VMs cannot sign in” nor “duplicate a logged-in environment transparently” is sound. [macOS virtualization](https://developer.apple.com/documentation/virtualization/virtualize-macos-on-a-mac?changes=_4_9&language=objc), [iCloud in VMs](https://developer.apple.com/documentation/virtualization/using-icloud-with-macos-virtual-machines?changes=_7)

Prefer a guest per **interaction domain**, not per crop. A media editor and its plugins may need one coherent session. Two independently edited contexts may need separate app instances or guests, but could still collide through the same server account. Guest display remoting, host/guest file semantics, audio, GPU fidelity and latency require measurements.

## 5. Workflow consequence

Consider a research-to-design workflow: a user selects an object in a native editor; eversion resolves its actor identity, updates a native property inspector, opens supporting web evidence and asks an agent to propose variants. While the user edits the current variant, background computation produces candidates through semantic commands or an isolated actor.

Accepting a candidate checks the original revision and applies a verified edit. If the source changed, eversion presents a rebase/merge choice rather than clicking stale coordinates. If a plugin opens a modal dialog, that actor’s dependent branch pauses while unrelated branches continue. A new target-app release invalidates only the incompatible adapter capabilities; source surfaces and workflow history remain available.

The agent’s contribution is not only clicking: it proposes typed operations, learns observed transition contracts, generates adapter candidates and constructs new workflow branches. Promotion requires reproducible evidence. An agent-observed action is initially a hypothesis; successful replay against one screen does not certify arbitrary contexts.

## 6. Five experiments that can overturn this design

1. **Input independence:** on a controlled fixture set spanning AppKit, Qt, SwiftUI and custom rendering, alternate text/IME, drag, scroll, menus, sheets and shortcuts between two surfaces while a separate host field receives user input. Record destination, focus theft, lost events and wrong-document effects. A single silent wrong-target mutation disqualifies that backend from unattended parallel input; scope its lease more broadly.

2. **Framework export fidelity:** export a rich text view and a Qt Quick scene while retaining the original model. Exercise resize, display-scale changes, popups, IME, undo, accessibility and object destruction. Compare source state and rendered behavior. If generalized framework handling repeatedly requires application-specific surgery, narrow the promised adapter family; if probe export adds no usable control over AX, stop prioritizing transplantation.

3. **Admission and identity:** in a separate authorized lab, test original signed apps, official plugin admission, cooperative builds and managed copies across two supported OS versions. Measure probe admission, startup, document opening, authentication, entitlements and updating. If useful coverage depends predominantly on altered identity or security weakening, retain probes as an optional specialist backend.

4. **Capture/lifecycle resilience:** test occluded, minimized, hidden, different-Space and suspended sources; resize and destroy/recreate windows during commands. Require every stale geometry epoch or dead object to reject/refresh before actuation. Any undetectable stale frame that can cause a wrong command defeats the “live component” promise for that capture mode.

5. **Isolation payoff:** run the same evolving workflow in ordinary actors and two independent guests with synthetic accounts/data. Measure input independence, memory, p95 visible response latency, GPU/audio fidelity, restart recovery and account reauthentication. If isolation cannot meet a declared interaction budget or preserve needed application functions, reserve it for asynchronous work instead of pretending it supports continuous direct manipulation.

These are proposed tests, not completed validation. The central unknown is whether framework-level adapters provide enough reusable behavioral control across real apps to justify their maintenance cost. The architecture should survive a negative answer by retaining owned workflow semantics and explicit weaker backends.
