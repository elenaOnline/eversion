> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Composing existing desktop applications: mechanisms and boundaries

Research date: 2026-10-07. Scope: existing third-party desktop applications, no source code or new vendor cooperation assumed. This is a technical feasibility analysis, not a claim that any combination has been tested. Linked API documentation and project source are primary sources; proposed architectures and consequences are identified as engineering synthesis.

## Bottom line

On macOS, a convincing, user-editable workspace containing live, interactable pieces of unrelated desktop apps is technically feasible as **capture + input/semantic proxies**. It need not merely be screenshots. However, that does not transplant the apps' internal controls, execution state, or data models into the host. There is no general documented API in the researched public AppKit/ScreenCaptureKit/Accessibility interfaces that takes an arbitrary foreign NSView and adopts it as one's own child control. Treat this as an API-surface conclusion, not an impossibility theorem about reverse engineering.

Windows and X11 have genuine cross-process window reparenting, which strengthens the embedding option. A Wayland compositor can control composition and input at the right architectural layer; an ordinary Wayland client deliberately cannot access other clients' surfaces. On every platform, combining **semantics** (selected record, shared search, cross-app undo, one transaction, one identity) requires a further adapter layer. Spatially putting live surfaces together does not create that layer.

The best Mac-first architecture is a hybrid: an unsandboxed, signed/notarized local broker; GPU-rendered window crops; Accessibility-based targeting; native reconstructed controls for reliable common actions; application-specific scripting/API adapters when already available; and a visible escape hatch to the original application. Do not make private WindowServer injection or security weakening a requirement for the basic product.

## 1. Four importantly different meanings of composition

1. **Visual collage:** independently sized, live crops of existing windows. All panes can update at once. No input forwarding is required.
2. **Interactive portals:** a crop forwards pointer/keyboard events or invokes an Accessibility action on the original app. The original process retains its controls and state. This can feel like one application to a human, although focus, menus, popovers, and modal dialogs remain foreign-app concerns.
3. **Actual foreign-window embedding:** the OS window hierarchy is changed (Windows HWND, X11 Window). Rendering and event handling remain in the original process. This is more direct than video capture but still not a shared application model.
4. **Semantic recomposition:** the host extracts values/objects/actions and builds new controls and workflows around them. This can genuinely rearrange buttons and fields independently, but only for functionality the adapter understands and can observe/control. UI automation, existing IPC, runtime instrumentation, or reverse-engineered application internals may supply the adapter.

These can coexist. For example, an editor canvas can remain a live portal, a selected object's properties can be reconstructed as native controls, and a separate application's search panel can be connected by an explicit dataflow rule.

## 2. macOS public mechanisms

### ScreenCaptureKit: reusable pixels, not controls

SCContentFilter can select a desktop-independent window, a set of windows, or applications on a display. Its inclusion/exclusion filters let the compositor omit the host itself and irrelevant content. [Apple SCContentFilter](https://developer.apple.com/documentation/screencapturekit/sccontentfilter/init%28display%3Aincluding%3Aexceptingwindows%3A%29?language=objc)

The capture stream yields sample buffers with IOSurface-backed CVPixelBuffers. Apple's sample obtains the IOSurface and places it into an NSView's layer; frame metadata includes content rectangle, scale, and status. Thus a native Metal/Core Animation scene graph can display several independent sources without video encoding or a network round trip. This is the right foundation for a low-latency local portal. It is not access to the source's NSView hierarchy. [Apple sample](https://developer.apple.com/documentation/screencapturekit/capturing-screen-content-in-macos?language=objc), [screen output](https://developer.apple.com/documentation/screencapturekit/scstreamoutputtype/screen)

Important implementation detail: Apple's sourceRect documentation says it is ignored when capturing a single window, whose full bounds are captured. **Crop the resulting single-window texture in the host**; do not assume sourceRect supplies arbitrary element-level single-window capture. Display capture does support a source rectangle. [Apple sourceRect](https://developer.apple.com/documentation/screencapturekit/scstreamconfiguration/sourcerect?language=objc)

Engineering synthesis: retain one stream per source window, share its latest texture among all crops, and store each crop in source-relative coordinates plus a semantic anchor where possible. Recompute transforms when window geometry, backing scale, content rectangle, or selected UI layout changes. Separate capture rate from host animation rate. Never display a stale image as if it proves a current action succeeded. Capturing a covered window and forcing a minimized/hidden application to continue drawing are different problems; test occlusion, minimization, app hiding, Spaces, full-screen mode, and background throttling independently on each supported OS/app.

### Accessibility: the most useful generic semantic layer

AXUIElement exposes the UI tree, roles, attributes, supported actions, and writable values. A host can address a button, set a field when settable, retrieve selected text, or obtain a control's bounds. AXObserver notifications provide change signals where the target implements them; registration can explicitly return notification-unsupported. This is a capability negotiation API, not a universal complete DOM. [Apple action and attribute API listing](https://developer.apple.com/documentation/applicationservices/1462057-axuielementpostkeyboardevent), [Apple AXObserverAddNotification](https://developer.apple.com/documentation/applicationservices/1462089-axobserveraddnotification)

Hammerspoon's documented AX wrapper is a concrete implementation reference. It emphasizes that only exposed objects/features are accessible, that references can become invalid, and that element-at-position depends on z-order; application-scoped lookup can constrain hit-testing. [Hammerspoon AX docs](https://www.hammerspoon.org/docs/hs.axuielement.html)

Engineering synthesis: users can select a foreign control and save a binding that combines app bundle ID, window/document identity, AX role/identifier, ancestry, label, and fallback geometry. Reacquire it after target recreation. Reconstruct small native controls with bidirectional AX bindings instead of forwarding every click. However, absent identifiers, virtualized lists, unlabeled canvases, custom GPU widgets, or incomplete notifications need app-specific logic, polling, or visual inference. A generic accessibility tree is not the application's full data model or business logic.

### CGEvent and keyboard targeting: interaction is feasible but must be engineered

CGEvent supports mouse/keyboard/scroll events and posting to the event stream or a target PID. Apple's AXUIElementPostKeyboardEvent explicitly supports posting to a specified application rather than always the active one. These are real building blocks for remote-like interaction with a crop. [Apple CGEvent](https://developer.apple.com/documentation/coregraphics/cgevent), [Apple targeted AX keyboard events](https://developer.apple.com/documentation/applicationservices/1462057-axuielementpostkeyboardevent)

Engineering synthesis: inverse-map portal coordinates into source coordinates and route a complete gesture to the same target. Prefer AX actions for buttons and editable values; use synthesized events for canvases or richer gestures. PID targeting is not a promise that every application will accept complete background interaction. Focus, target window selection, first responder, menu tracking, input methods, key equivalents, secure fields, pointer capture, and modal state remain app-dependent. Do not equate “can post an event” with “has all independent input seats.” A host must explicitly own logical focus and reconcile it with OS/app focus. Several panes can remain live and interactable by turns, just as ordinary controls in one app do. Truly independent simultaneous keyboards/pointers may require separate sessions or specialized compositor support.

Two different crops from the same source window also share its scroll/selection/modal state. Cropping is not cloning. If two panes must display independent positions or independent documents, use distinct source windows/instances or rebuild the view from semantic data.

### Existing scripting and IPC: meaningful functions without transplanting UI

Scripting Bridge dynamically bridges an application's existing scripting definition and Apple-event object model, exposing properties, collections, and commands. It can control and exchange data with scriptable applications; it does not make an unscriptable application scriptable. [Apple Scripting Bridge](https://developer.apple.com/documentation/scriptingbridge)

Already available AppleScript dictionaries, URL schemes, command-line interfaces, local sockets, plugins, and vendor APIs should outrank coordinate automation. They permit a newly designed interface to invoke durable operations instead of reproducing an app's screen. XPC alone is not a generic inspector of another program; a service and protocol must already exist or be added inside a cooperating/instrumented target.

Apple Events have their own privacy declaration and entitlement requirements. NSAppleEventsUsageDescription is required for APIs that send them, and the automation entitlement permits requesting authorization. This access may expose sensitive data through the target even where direct file access is denied. [Apple usage description](https://developer.apple.com/documentation/bundleresources/information-property-list/nsappleeventsusagedescription), [Apple security entitlements](https://developer.apple.com/documentation/bundleresources/security-entitlements)

### Overlays and arranging real windows

A less invasive variant arranges original windows and paints host chrome around or over them. User input can reach actual source surfaces, avoiding forwarding in exposed areas. This preserves original interactive behavior but leaves window geometry, z-order, activation, Spaces, popups, and clipping difficult to make coherent. Painting a crop-shaped hole is not arbitrary reparenting, and moving a foreign top-level window is not adopting its internal child views.

Amethyst demonstrates public Accessibility-driven window management. Hammerspoon combines AX, event taps, and programmable UI. These are useful construction tools, not products that already provide arbitrary control extraction. [Amethyst](https://github.com/ianyh/Amethyst), [Hammerspoon modules](https://www.hammerspoon.org/docs/index.html)

## 3. Permissions, distribution, and private mechanisms

### TCC and Mac App Store are architectural constraints

Capture and control are separate grants. SCContentSharingPicker lets a person authorize selected content for a capture session without separately granting broad screen recording access, and macOS shows a sharing indicator. It does not grant Accessibility control or Apple-event automation. [Apple privacy session](https://developer-rno.apple.com/videos/play/wwdc2023/10053/), [SCContentSharingPicker](https://developer.apple.com/documentation/screencapturekit/sccontentsharingpicker)

Apple's current sandbox documentation explicitly lists using Accessibility APIs in assistive apps and sending Apple Events to arbitrary apps as incompatible with App Sandbox. An Apple DTS answer reiterates that sandboxed apps cannot use the Accessibility APIs and that the Mac App Store requires sandboxing. Do not confuse exposing one's own app accessibly (allowed and encouraged) with controlling other apps as an AX client. The generic broker therefore belongs in a directly distributed unsandboxed app, not an assumed normal Mac App Store configuration. [Apple sandbox restrictions](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox), [Apple DTS answer](https://developer.apple.com/forums/thread/789663)

App Store guideline 2.5.1 requires public APIs. Direct distribution and notarization are distinct from App Store approval, but neither abolishes TCC, SIP, signing requirements, target-process protections, or protected-content behavior. [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)

### Private WindowServer/SkyLight APIs

Yabai is a strong real-world reference for the boundary between ordinary AX window management and privileged window-server manipulation. Its repository explains that optional Dock injection requires partially disabling SIP to reach additional privileged operations. Its changelog documents repeated OS-version-specific fixes for scripting additions and SkyLight behavior. Recent features have moved back to SIP-enabled paths, so **do not claim that all advanced window operations inherently require disabling SIP**. Classify each operation and tested OS version separately. [yabai repository](https://github.com/asmvik/yabai), [yabai changelog](https://github.com/asmvik/yabai/blob/master/CHANGELOG.md)

Engineering synthesis: private APIs may enable transforms, layers, space manipulation, or lower-level surface handling that make a portal more seamless. They remain undocumented compatibility dependencies, not a stable public component model. Additional drawing access does not, by itself, merge semantic models or create independent app focus. An architecture should degrade to public capture/AX rather than depend on Dock/WindowServer patching.

### Virtual displays and remote sessions

DeskPad creates a virtual monitor mirrored into its app window. The source tree includes CGVirtualDisplayPrivate.h: useful evidence that virtual-display workspace designs exist, but are not necessarily public-API-only. A virtual display can keep source apps in a known staging area rather than behind the user's visible working windows. It is still one macOS session with its focus/keyboard/menu constraints. [DeskPad README](https://github.com/Stengo/DeskPad), [DeskPad source tree](https://github.com/Stengo/DeskPad/tree/main/DeskPad)

A remote/VM/session-per-application variant isolates focus and lets the host compose local render streams. This is an engineering option, not free extraction of the currently running local app: it introduces application installation, licensing, login/state migration, GPU/audio/peripheral, file access, clipboard, and latency concerns. A macOS virtual display does not automatically create another login session. Separate execution may also change app behavior and entitlement support.

### Native injection / runtime patches

For a willing owner of a compatible app, loading a mod in-process can expose private objects, change layout, intercept actions, or export a richer semantic bridge. It is a materially different tier from screen capture. The obstacle is not simply time: hardened runtime uses library validation and DYLD protections to resist injection. Setting exceptions in the **host's** entitlements does not grant permission to inject into an unrelated target. Re-signing/patching the target can alter its identity and compatibility. [Apple notarization and hardened runtime explanation](https://developer-rno.apple.com/videos/play/wwdc2019/703/)

MacForge/SIMBL-style plugins are a historical example of modifying existing apps without their source. The project's published installation instructions require security weakening; its public product page is old and explicitly says it lacks M1 support. Treat it as a mechanism/example, not a verified current Apple Silicon solution. [MacForge](https://www.macenhance.com/macforge.html), [MacForge installation](https://github.com/MacEnhance/MacForge/wiki/Installation)

Windows Windhawk is a more active example: its source documents the global code-injection/hooking approach and process compatibility concerns. BepInEx similarly demonstrates runtime patching and plugins for Unity/.NET targets, with runtime/platform-specific support rather than universality. Neither is a generic cross-app composition framework, but both show how a runtime-specific adapter can access more than pixels. [Windhawk](https://github.com/ramensoftware/windhawk/blob/main/README.md), [Windhawk injection targets](https://github.com/ramensoftware/windhawk/wiki/Injection-targets-and-critical-system-processes), [BepInEx](https://github.com/BepInEx/BepInEx)

## 4. Windows: real HWND embedding plus compositor replicas

SetParent can change a pop-up, overlapped, or child window's parent and explicitly documents cross-process use. Style flags must be adjusted separately, UI state synchronized, and DPI differences may reset the child process's DPI awareness. A host can therefore genuinely contain a foreign HWND, including a region implemented as its own child HWND. A canvas-drawn button with no independent HWND does not thereby become extractable. [Microsoft SetParent](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setparent)

Engineering synthesis: reparenting an entire top-level window inside a clipped viewport is a plausible desktop portal with original event handling; extracting an individual control can break parent notifications, layout, message routing, or assumptions about its hierarchy. App-specific modern UI frameworks often use fewer native child windows, reducing the granularity. Track ownership, destruction, popups, IME, drag/drop, accelerators, accessibility, and process exit explicitly.

DWM thumbnails offer live source-window replicas with source/destination rectangle control. They are a presentation mechanism, not interactive controls. OnTopReplica is a concrete open-source example combining them with region selection and click forwarding. Its issue tracker records the classic consequence: forwarded clicks can activate and raise the original window. [DWM thumbnails](https://learn.microsoft.com/en-us/windows/win32/dwm/thumbnail-ovw), [OnTopReplica README](https://github.com/LorenzCK/OnTopReplica/blob/master/README.md), [focus issue](https://github.com/LorenzCK/OnTopReplica/issues/65)

UI Automation control patterns provide semantic actions such as invoke/value/selection rather than HWND dependence; as on Mac, provider support limits what can be reconstructed. SendInput is restricted by UIPI to equal/lower integrity levels, so an ordinary desktop app cannot assume it controls elevated/security-sensitive apps. [UIA control patterns](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-controlpatternsoverview), [SendInput](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput)

Windows has explicit capture exclusion through SetWindowDisplayAffinity. This API is not itself DRM and Microsoft does not describe it as perfect protection, but a compositor cannot assume every visible window will be available in its capture pipeline. [Microsoft display affinity](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowdisplayaffinity)

## 5. Linux: X11 and Wayland have opposite default access models

XReparentWindow genuinely inserts a window under a new parent on the same screen, including unmap/remap and hierarchy notifications. Save-set functions protect a foreign window from destruction if the embedding/window-manager client exits. This makes X11 a very permissive laboratory for actual foreign-window composition. Granularity still depends on whether a desired control is a real X window, and application hierarchy assumptions still matter. [Xlib specification, chapter 9](https://www.x.org/releases/X11R7.5/doc/libX11/libX11.html)

Wayland explicitly says ordinary clients cannot access other clients' surfaces or know their global positions. This rules out assuming X11-style arbitrary reparenting from a normal application. But the compositor itself receives buffers, decides the scene graph, transforms input coordinates, and routes input to clients. Therefore a custom/nested compositor is the most principled way to provide arbitrary visual transformations and input routing for apps launched in that environment. It does not extract internal widgets or semantic data. [Wayland protocol model](https://wayland.freedesktop.org/docs/book/Protocol.html), [Wayland architecture](https://wayland.freedesktop.org/docs/book/Architecture.html)

For an ordinary app, the ScreenCast/RemoteDesktop portals provide an authorized capture-and-control route through PipeWire plus input injection (recommended EIS/libei, or D-Bus notification methods). The portal presents a user selection/permission dialog and returns granted devices/streams; desktop/backend support must be checked. [XDG RemoteDesktop portal](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.RemoteDesktop.html)

Xpra demonstrates remote individual-app windows, persistent sessions, and clipboard/audio integration. Its current documentation states seamless servers are unavailable on macOS/Windows, where shadow mode shares an existing desktop; X11 is the broad-compatibility server backend and the Wayland backend is experimental. Thus “use Xpra” is a genuine Linux-app composition route, not a ready-made arbitrary Mac-app extraction answer. [Xpra seamless mode](https://github.com/Xpra-org/xpra/blob/master/docs/Usage/Seamless.md)

## 6. Recommended architecture and decisive experiments

This section is design synthesis, not a claim about an existing complete product.

### Three cooperating planes

- **Surface plane:** selected source windows → capture streams → GPU crops → user-editable layout. Keep state indicators, source identity, stale-frame state, and a direct-open-original affordance.
- **Action plane:** capability router chooses existing API/scripting operation, then AX/UIA action, then fully tracked event gesture. Escalate to the original app for unsupported/secure interactions. Keep irreversible actions explicit.
- **Semantic plane:** adapters declare typed inputs/outputs, stable entity IDs, subscriptions, allowed mutations, provenance, and conflict policy. Shared search/selection/workflow rules live here. AI can help infer or author adapters but cannot replace missing authority or prove an action succeeded from a plausible image alone.

### Interaction contract

A pane must declare whether it is view-only, remote-interactive, semantic/reconstructed, or native-embedded. Do not silently substitute screenshots for unavailable controls. Modal dialogs and popovers must be surfaced as owned auxiliary portals or opened in the original app. Bindings must identify the document/window, not just the first process with a matching title. Keep app-local undo separate unless a semantic adapter genuinely implements coordinated undo.

### Small prototype that would resolve the largest uncertainty

Use three heterogeneous Mac targets: an accessible AppKit text editor, a Chromium/Electron application, and a custom-rendered canvas application. For each, test:

1. Multiple live crops, covered source windows, changing scale/size, another Space, minimized/hidden states, source relaunch, screen lock/unlock
2. AX-selected button activation and field editing versus CGEvent-routed mouse/keyboard/scroll/drag; compare background input and controlled activation
3. Popovers, menus, sheets, context menus, IME/dead keys, clipboard, drag/drop, secure fields, keyboard shortcuts
4. Two crops sharing one source window's state versus two distinct source windows
5. A reconstructed control bound to the source plus one explicit cross-app semantic rule; verify outcomes through authoritative state
6. Revoked capture/AX/automation permission; graceful removal of privileges; no claim of success from stale content
7. Performance under many streams, source stalls, display sleep, HDR/SDR, memory pressure, target crashes

The key product bet is not whether pictures can be composited; public APIs and existing projects already establish that. It is whether an input/focus broker and a small set of semantic adapters can make the chosen workflows feel local and reliable while clearly handling the boundaries. With arbitrarily large engineering resources, per-app adaptation can be broad; blanket “arbitrary app, arbitrary control, fully native and semantically unified, no cooperation, no permissions, always survives upgrades” remains a different and unsupported promise.

## 7. Hard cases: what can be solved, and what changes the architecture

The ambitious target is a **retained-runtime composition system**: keep original applications alive as engines while making their useful surfaces/actions available in a new workspace. Capture is a transport, not necessarily the end-user abstraction. The following are design consequences of that model.

### Independently scrollable slices from one source window

A source scroll view has one scroll position. Two simultaneous texture crops of it cannot show two independently changing positions merely by assigning different host scrollbars. Options are: (a) two source windows/views if the app supports them; (b) separate instances/profiles/sessions, with document/account state synchronization explicitly handled; (c) a semantic replica with its own viewport over retrieved data; or (d) app-specific instrumentation that creates another internal view. A large captured backing image that the host scrolls is useful only while its data remains complete/current, and does not supply original offscreen controls. Repeatedly scroll-capture-scroll-back is time-multiplexed emulation, with side effects and races, not two independent live views. Virtual displays create staging room; they do not duplicate one scroll view's state.

### Keyboard focus, composition, and candidate windows

The host can maintain a logical focused portal, but the original app still maintains its first responder and active text-input context. AppKit binds input methods to the first responder in the key window and asks the text-input client for character/caret rectangles to place candidate UI. [Apple text editing architecture](https://developer-rno.apple.com/library/archive/documentation/TextFonts/Conceptual/CocoaTextArchitecture/TextEditing/TextEditing.html)

Therefore, key forwarding alone is insufficient for a polished global IME solution. Choices: keep the original responder active and capture/position all related candidate UI; route text through a host-native input control and commit into the source through a verified adapter; or implement a fuller text-input bridge including composition, selection, replacement ranges, geometry, and cancellation. Committing Unicode is not equivalent to preserving the source's incremental marked-text semantics. Separate sessions isolate responder state but still require an IME/session bridge to the host.

### Popovers, menus, sheets, and drag feedback outside the crop

These may be separate windows, part of a larger parent texture, or system-owned UI. A fixed rectangle loses them. A retained-runtime system should discover auxiliary surfaces, attach them to the logical source pane, map their placement to host coordinates, and route modal focus until dismissed. Context menus may appear on mouse-down and change normal event delivery; a drag is a stateful gesture with capture and drop negotiation, not a series of independent clicks. The exact association is app/framework-dependent. Cropping must not suppress consequential prompts or make an app appear unresponsive while an off-crop modal dialog is active.

### Open/save panels and security-grant semantics

Apple documents that Open panels have been drawn in a separate process since macOS 10.15 regardless of sandboxing, and that choosing a file adds access to the requesting app's sandbox. [Apple NSOpenPanel](https://developer.apple.com/documentation/AppKit/NSOpenPanel?language=objc)

A compositor cannot simply assume an app-only capture includes its entire open/save flow. It must identify and present the real dialog and its ownership. Replacing that with the host's file picker selects for the host; passing a path is not automatically the same sandbox grant to the target. A supported target protocol for opening user-selected files, an explicit transfer, or the original dialog is needed. Privileged authorization/secure-entry prompts should remain unmistakably genuine OS UI; an ambitious seamless workspace must have a security boundary rather than imitate those prompts.

### Hidden/minimized sources, app suspension, and frozen pixels

No capture API can create a frame the application never rendered. The broker should distinguish unchanged content from blank/stopped capture; ScreenCaptureKit provides explicit complete, idle, blank, suspended, and stopped states. [Apple SCFrameStatus](https://developer.apple.com/documentation/screencapturekit/scframestatus)

A staging display with unminimized windows, a controlled separate session, or an application-specific keep-rendering setting can reduce freezes. None is a universal guarantee against application background throttling or session locking. Preserve the last frame as a clearly stale preview, probe liveness separately, reconnect streams when necessary, and reacquire source identities after topology changes. Retaining the runtime is an asset, but the compositor must supervise it.

### The host's own accessibility tree

A video texture does not automatically inherit the source control's accessibility hierarchy. The host needs a meaningful tree at the **new** positions: projected AX nodes with transformed/clipped geometry and forwarded actions, or native reconstructed accessible controls. Apple provides NSAccessibilityElement specifically for UI objects without standard backing views, including role, parent, frame, and notification support. [Apple NSAccessibilityElement](https://developer.apple.com/documentation/appkit/nsaccessibilityelement-swift.class?language=objc), [Apple accessibility integration sample](https://developer.apple.com/documentation/Accessibility/integrating-accessibility-into-your-app)

Engineering consequences: avoid duplicate exposed controls when the original source also remains in the desktop's accessibility tree; define which pane owns keyboard/VoiceOver focus; map text ranges and caret geometry as well as rectangles; update notifications when sources change; do not announce inferred labels as authoritative data. A crop of a source canvas with no AX children may need newly authored descriptions and controls. This is solvable product work, but it is part of the composition platform itself, not a free benefit of AX permission.

### Recovery and persistence

Persist the workspace's source bindings, document identity, crop/semantic anchors, layout, permissions needed, and adapter versions. Do not persist transient window IDs as the sole identity. On app exit or crash: mark unavailable, preserve layout, reconnect to the intended document on explicit/authorized relaunch, and only replay operations known to be safe/idempotent. Native app crash recovery and unsaved state remain that app's responsibility unless an adapter supplies a real checkpoint. A captured texture cannot recover unsaved application state. Never replay a pending save/send/purchase merely because the screen went blank before a success response.

### Why no generic full-model extraction follows from these tools

The limits are informational and contractual as well as engineering. A runtime can expose pixels and some UI actions while retaining undisclosed business rules, server-side operations, hidden data, identity requirements, and state transitions. A compositor can keep that runtime as the authoritative engine and achieve extremely broad practical reuse. It cannot infer a complete, independent, equivalent model from one visible interaction surface by API composition alone. Runtime instrumentation, reverse engineering, or cooperative protocols can expand the model per target, and still do not remove remote authorization or licensing constraints. This leaves substantial room for an ambitious product: universal surface hosting, a capable input/session layer, richly accessible projected interfaces, and progressively stronger semantic adapters, with honest capability boundaries.
