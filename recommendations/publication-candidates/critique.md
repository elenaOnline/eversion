> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Critical review: do not shrink the idea to screen automation

Research date: 2026-10-07. Reviewed native.md, semantics.md and modding.md. Read-only primary-source research; no application instrumentation, installation or interaction was performed. The proposed architecture and acceptance criteria are analysis, not a claim of a working implementation.

## Main judgment

The strongest credible interpretation is a **distributed component host for already-running application engines**. A component may keep its model, command handlers, view objects and renderer inside the source process while publishing a remote control/view interface. Its presentation can be original pixels, an embedded native surface, an in-process rearranged view, or a new host-rendered frontend. These are interchangeable implementation choices within one product, not a ladder in which only a physically transplanted widget is “real.”

The core idea is feasible in substantial cases without original source or fresh vendor cooperation. It remains unproven as an automatic general product. The appropriate open question is how broadly runtime discovery and adapter synthesis can create reliable component contracts under allowed access, not whether a standard universal API exists today.

The native memo is strongest on mechanisms and permission boundaries. The semantic memo is strongest on state fidelity. The modding memo supplies the ambitious missing middle: source-independent framework introspection and runtime adaptation. Lead with that synthesis; use the constraints to specify the product, not to quietly replace the request with a dashboard.

## 1. Four distinctions that change the conclusion

### Source-independent is not application-independent

“No predetermined stack” can mean the system begins without knowing whether an app uses Qt, Cocoa, WPF, Chromium or a custom engine. It may identify the stack during discovery and select or synthesize the corresponding probe. This satisfies source-independent discovery. Requiring one unchanged adapter for all applications is a much stronger condition the user need not have intended.

Similarly, no vendor cooperation does not forbid using an existing accessibility implementation, plugin interface, scripting dictionary or debugging facility. It forbids requiring the vendor to redesign its product for this experiment. Existing hooks should be used when present, but they should not be the only evidence offered for the ambitious case.

### Open-ended target selection is not guaranteed universality

An open-ended tool can accept a previously unsupported app, inspect it, generate an integration and report a genuine blocker when necessary. That is meaningfully different from a closed whitelist even if it cannot support every protected process, server-side function or input mode. Do not answer an open-world product idea solely by disproving a universal quantifier.

### Rendering remotely does not make a component fake

A remotely rendered component can expose exact domain actions and subscriptions. Conversely, a locally recreated control can be little more than a guessed coordinate macro. Judge by authority, state identity, deterministic interaction and lifecycle, not by whether the host sees a texture or a widget pointer.

A pixel portal is a useful rendering method. A product made only of portals with no reusable bindings is narrower than semantic composition. Both statements can be true without dismissing the first as “just screenshots.”

### Shared undo and global transactions are optional stronger features

A real composed application can have independent source undo stacks, explicit cross-app actions and non-atomic workflows. Existing applications already contain plugins, external services and operations that do not share one transaction. Global undo is valuable, but failure to guarantee it does not refute ordinary composition. Label guarantees per operation rather than making perfect global consistency an entrance requirement.

## 2. Unavailable public APIs are not impossibility proofs

Frida documents function interception, calling native functions, Objective-C runtime access and implementation replacement. This is a concrete mechanism for exposing application behavior beyond pixels or accessibility. It does not automatically identify safe domain semantics, and only applies where execution/instrumentation is actually permitted. [Frida JavaScript API](https://frida.re/docs/javascript-api/)

Chrome's debugger extension API can inspect JavaScript, mutate DOM/CSS and instrument network interactions. Its exposed protocol domains include Runtime, Debugger, Input, DOM and Accessibility. It also documents enterprise restrictions that can refuse attachment. This is a much richer authorized channel than DOM scraping alone. [Chrome debugger API](https://developer.chrome.com/docs/extensions/reference/api/debugger)

The inference is important: a missing public “selected document object” API may be recoverable by observing the runtime object and command that already implement selection. Richer instrumentation can invalidate an earlier claim of unobservability. A blank accessibility tree is evidence against an AX-only adapter; it is not evidence that no semantics can be recovered.

The precise limit is: two states indistinguishable through **every permitted channel available to the implementation** cannot be distinguished reliably by that implementation. That qualifier prevents a valid information-theoretic claim from doing too much rhetorical work.

Equally, do not overcorrect into “all semantics are in memory.” Server-only objects, inaccessible account data and remote commit state may remain unavailable. A client can have a pending optimistic update without knowing whether the server durably committed it. Those are genuine authority/observation boundaries.

## 3. Native input: plausible hosting, not yet demonstrated transparency

### The public Mac primitive is real

Apple explicitly documents AXUIElementPostKeyboardEvent as targeting a specified application rather than always the active application. It accepts an application or system-wide accessibility object; it does not accept an arbitrary control as the target. CGEvent also exposes postToPid. These make an interactive portal credible, but neither API promises a complete independent input session for each crop. [AX targeted keyboard input](https://developer.apple.com/documentation/applicationservices/1462057-axuielementpostkeyboardevent), [CGEvent](https://developer.apple.com/documentation/coregraphics/cgevent)

AppKit routes ordinary keyboard events through key equivalents, interface navigation, the key window and first responder. Mouse handling has its own hit-testing and gesture ownership. Target/action dispatch can take another responder path. Therefore “the event reached the PID” is a weaker success condition than “the intended component behaved exactly as if hosted normally.” The archived architecture guide explains these dependencies; modern behavior must still be measured. [Apple Event Architecture](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/EventOverview/EventArchitecture/EventArchitecture.html)

### Shared session is not inherently fatal

A normal single-user composed workspace only requires one active keyboard/text focus at a time. It can still show many continuously updating components. Requiring independent simultaneous keyboards, several unrelated IME composition sessions or parallel synthetic drags imposes a much stronger specification than ordinary desktop use.

The host should own a logical focus path: workspace → component → source window/view. A full pointer gesture remains attached to the selected component even when the pointer crosses its edge. Shortcut resolution follows the selected component except for explicit host shortcuts. Menus/popovers belong to that component and must appear at translated host geometry. Switching components must cancel or complete text composition correctly, not merely redirect raw keycodes.

Without an in-process probe, some apps may need actual activation, foreground window changes or an escape to the original app. That is a fidelity tier, not a hidden success. Capture cannot guarantee an invisible source window keeps rendering; test visible, covered, minimized, hidden and background cases separately.

### The stronger authorized approach

An in-process adapter can dispatch through the source framework's actual view/action mechanism, observe popup creation, expose text-input state, and report component focus independently of the host's physical window. In principle this permits richer integration than OS event forwarding. It must preserve whatever global assumptions the source app has, including singleton selection, active-document variables and modal loops. Ignoring engineering difficulty makes such virtualization worth exploring; it does not prove it works for every application.

Windows' AttachThreadInput provides a concrete example of input state being shared across threads, including current focus. It has constraints and resets key state when called. This is machinery for solving input integration, not a generic complete solution. [Microsoft AttachThreadInput](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-attachthreadinput)

Qt's window-container documentation similarly describes embedding with platform-dependent focus activation, focus-return responsibilities, stacking and rendering limitations. Even cooperative framework embedding has an input contract to implement. This strengthens the case for explicit component focus/lifecycle interfaces rather than assuming reparenting solves everything. [Qt QWidget/window containers](https://doc.qt.io/qt-6.8/qwidget.html)

**Conclusion:** transparent native hosting is a credible target architecture with authorized runtime adapters. The researched sources establish primitives and failure modes, not a tested universal transparent host.

## 4. A concrete stronger architecture

The following is a proposed design.

### A. Discovery and qualification

Start from the user's selected app region/action. Discover process/runtime, visible component hierarchy, existing interfaces and actual permissions. Identify both view objects and candidate state/command owners. Prefer observing ordinary user actions in a disposable document and correlating them with property changes/events over guessing semantics from symbol names.

Generate a capability manifest that distinguishes exact domain state, displayed state, inferred state and opaque render surfaces. A failed probe should cause an explicit downgrade or refusal, never a claim that the visual result proves the binding is exact.

### B. Keep the engine and object ownership in place

Retain original execution, authentication, document model, plugin environment and backend communication. Export remote handles to selected objects/actions. The host should not copy an entire app's state merely to expose one tool.

Where possible, create a second view bound to the same model. Otherwise relocate/re-layout the existing view in its own process or host a live surface representing it. If the source only has one selection/viewport, the contract must say so. Duplicating a texture does not create independent viewport state; an independent view requires an actual source view or a semantically reconstructed one.

### C. Separate control, interaction and rendering paths

- Control path: typed domain reads, commands and subscriptions, with source identities and versions
- Interaction path: focus, selection, pointer gestures, keyboard/IME, menus, drag/drop and accessibility
- Rendering path: source surfaces, original framework rendering, or replacement host controls

GPU sharing optimizes rendering transport. IPC transports calls/events. Neither creates semantics, but neither prevents rich semantics from being transported once an adapter supplies them.

### D. A component is more than a screenshot and less than an entire app

The minimum contract should include identity, lifetime, sizing, state owner, supported actions, change events, logical focus, popup surfaces, accessibility representation, permission scope and disconnect behavior. A workflow binding adds explicit type/unit conversion and provenance. This makes a component reusable across several compositions without rerecording a click sequence each time.

### E. AI works mainly during discovery and repair

Use AI to hypothesize object meanings, trace relationships, generate adapter code, write tests and diagnose version drift. Steady-state interaction should use a deterministic adapter. This is a compiler/adapter-development role for AI, not a requirement that each keystroke wait for a model to understand a screenshot.

An integration catalog remains useful, but the distinguishing promise is the ability to create a new entry from an unfamiliar installed app. The key unknown is empirical adapter-generation reliability, not availability of an LLM orchestration envelope.

## 5. What a convincing demonstration must establish

### Do not let a convenient integration prove the wrong claim

Embedding Neovim via its intended remote interface demonstrates high-quality runtime reuse and helps validate the host contract. It does not establish source-independent extraction from an unsupported app. Likewise, an Ableton API example validates semantics where an existing interface supplies them. Those are excellent baselines, but at least one target must lack a prewritten adapter for the chosen feature.

Run two tracks:

1. **Host quality:** known rich interfaces test interaction, composition UX, latency and failure recovery without ambiguous extraction problems
2. **Adapter discovery:** an unfamiliar application/version and a previously unintegrated feature test the actual source-independent thesis

Neither alone proves both.

### Proposed acceptance test

Choose one native app and one website. Keep both originals running with real unsaved state. Select a meaningful region in each. Produce at least one component with a domain action/state subscription and one original live editing surface. Build a third control in the host that binds to source state, then use that same component in a second composition without a new recording.

Demonstrate:

- Changes made in the original immediately update the composed control, and vice versa, against the same underlying object
- A supported meaningful action runs directly through its discovered interface, not a brittle assumed click location
- Native typing, non-Latin IME input, selection, undo, shortcuts, scrolling, drag outside component bounds and a popup work correctly
- Source window resize, occlusion and replacement do not silently misroute input
- Two views either maintain independent viewport/selection as advertised or visibly declare shared state
- A cross-app selection-to-query or parameter-to-preview binding is reusable and preserves source identity
- A new document/object, duplicate labels and an app restart either rebind correctly or stop clearly
- Permissions can be revoked without leakage, and a denied richer probe is not silently bypassed
- Source update testing distinguishes automatic repair from manual intervention and reports false-success rate

Global transactions and cross-app undo are follow-on tests if promised, not prerequisites for a first meaningful demonstration. Universal claims need diverse holdout apps and adversarial tests, not one impressive curated workflow.

## 6. Current factual checks

### Google Docs: prior categorical limitations are obsolete

The current official guide, updated September 30, 2026, documents comments, writing in suggestion mode, and accepting/rejecting/deleting suggestion threads. It also warns that comment/suggestion persistence may fail while document-model changes succeed. The semantics memo correctly reflects this. Do not quote older advice that the API cannot create suggestions or comments. [Google Docs comments and suggestions](https://developers.google.com/workspace/docs/api/how-tos/suggestions)

### Ableton: Link Audio is real in current documentation

The current manual explicitly distinguishes tempo/beat/phase synchronization from real-time Link Audio streaming among compatible peers. “Link never carries audio” is no longer correct. This does not imply transport of arbitrary Live project state or plugin internals. [Ableton Link and Link Audio](https://www.ableton.com/en/manual/synchronizing-with-link-tempo-follower-and-midi/)

### Mac sandbox Accessibility restriction is well supported

Apple's current sandbox documentation explicitly lists assistive Accessibility API use as incompatible. A June 2025 DTS answer reiterates sandbox incompatibility and the Mac App Store sandbox requirement. This supports a directly distributed unsandboxed broker for generic cross-app AX control. Do not confuse that restriction with whether an app may expose its own accessible UI. [Apple sandbox documentation](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox), [Apple DTS answer](https://developer.apple.com/forums/thread/789663)

### Chrome consent flow is specific, not a universal rule about every CDP connection

Chrome's active-session MCP auto-connect flow is documented for Chrome 144+, after the user enables remote debugging. Each connection request in that flow gets a permission dialog; an active session shows an automation banner. Other connection modes still exist. Say this is the consent behavior of the documented auto-connect route, not that every debugger transport has identical UI or silently covers every profile. [Chrome active-session debugging](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session)

## Recommended conclusion

Yes: there is a credible route to building an open-ended compositor of existing app capabilities by discovering and adapting their running implementations. The best architecture leaves engines where they are, exposes selected live state/actions, and composes views through a common host contract. Source access is not a prerequisite; runtime understanding and allowed instrumentation are.

The unresolved research/product challenge is reliable discovery and verification across heterogeneous apps, plus transparent interaction hosting. Security-denied access, server-only state and irreversible external effects remain real boundaries. Maintenance, binary/framework diversity and adapter generation are engineering problems under the user's premise, so they should shape the architecture and experiments rather than be used as a disguised “no.”
