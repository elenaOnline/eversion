> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Composing existing applications into malleable desktop software

Research and architectural assessment

Gen · 7 October 2026

## The conclusion

Yes. Your proposed app is technically viable in a substantially richer form than a collection of embedded websites or cropped windows. It could let someone take a useful part of an existing application, retain the real application behind it, place that part in a personally assembled workspace, and connect its state and actions to parts of other applications. Access to the original source code is not a prerequisite for every integration. Nor must every application author have anticipated the integration.

The architecture I would pursue is a **distributed component host**: the original applications remain running and retain the behavior people rely on; a local composition layer exposes selected surfaces, objects and operations through adapters. Some adapters use existing APIs. Others discover the running application's framework, inspect its object model, instrument permitted execution paths, or infer an interface from observable behavior. The user composes the resulting tools. An agent helps create and maintain the adapters.

Under your instruction to set aside engineering difficulty, the strongest answer is therefore more ambitious than “put screenshots on a canvas.” The interesting opportunity is to make existing software behave as reusable components, even when it was not distributed that way. The qualification is that **source-independent does not mean knowledge-free or adapter-free**. A system can discover the necessary knowledge after it encounters an app, rather than requiring the user to supply it in advance.

My assessment separates three claims:

- **Feasible and directly precedented:** recombining live, interactive regions of existing applications and websites without rewriting them
- **Feasible with application or framework adaptation:** exporting meaningful state and behavior, presenting new controls around it, and connecting it to other tools
- **Not an honest unconditional promise:** every arbitrary app and service yields all its hidden state, accepts every desired operation, and participates in lossless shared editing under every security policy

The last claim is considerably stronger than your product needs to be. The absence of universal undo, for example, does not prevent a genuinely useful composable application. Those boundaries should inform feature contracts rather than become an excuse to retreat from the idea.

This report is based on primary documentation, research papers and project source. It includes source inspection where a project's design description and current implementation differ. I did not install or execute these systems, modify your machine, or benchmark a prototype. Proposed architectures and product judgments below are my synthesis, not measured results. [Personal workflow reference omitted.]

## What the passthrough analogy actually establishes

I interpreted “pass through mods” as the recent family of cross-game integrations such as SkyCraft and Minecraft inside GTA. That is a strong architectural analogy, even if you had a different particular example in mind.

SkyCraft keeps Minecraft and Skyrim running. Its bridge connects their state, input and behavior rather than rebuilding both games inside a new engine. Importantly, the repository's draft design and current protocol should not be conflated: the inspected protocol contains block meshes, texture atlases, posed geometry, collision information and gameplay events. It is deeper than placing one game's video on top of another. The README also lists limitations and ownership conflicts, including features unavailable while Minecraft drives the player. [SkyCraft repository](https://github.com/chasmlol/SkyCraft) · [Current protocol](https://github.com/chasmlol/SkyCraft/blob/main/protocol/skycraft_protocol.h) · [Draft design](https://github.com/chasmlol/SkyCraft/blob/main/docs/DESIGN.md)

A separate GTA and Minecraft project uses a different implementation: local messages carry camera, input and ground information; shared memory carries image and depth layers; an add-on composites the output; gameplay events are translated into effects in the other world. The technique preserves both running programs while defining selected relationships between them. These are documented experimental projects, not independent evidence of universal compatibility. [Wither Storm passthrough](https://github.com/VortexisTV/wither-storm-gta5-passthrough)

For desktop software, the analogous bridge would carry document identities, selections, viewports, commands, property changes and rendered surfaces instead of camera transforms and collision geometry. An original application could supply its editor, computation engine or plugin behavior while another supplies a browser, inspector or research surface. A new host supplies the arrangement and connections.

The lesson is not that sufficiently fast IPC automatically makes two applications compatible. The impressive part is defining which runtime owns what and translating the particular behavior that crosses between them. The desktop version needs the same discipline, but can make that adapter construction progressively more automatic.

## What exactly can be composed

It helps to distinguish the things people mean by a “piece of an app.” A universal-looking interface can conceal very different capabilities.

| Piece | What is retained | What the host can do | Main condition |
|---|---|---|---|
| Live visual region | Original rendering | Move, scale, arrange, observe | Capture permission and continued rendering |
| Interactive portal | Original rendering and behavior | Route gestures or semantic actions to the source | Correct focus, coordinates, popup and lifecycle handling |
| Exported control | Original command or property | Present a different control bound to the same behavior | A verified action and state interface |
| Domain object | Application-specific meaning | Link a track, document range, record or selection | Identity, type and mutation semantics |
| Hosted engine | Original computation runtime | Build a substantially new frontend around it | A sufficiently rich protocol or runtime adapter |
| Shared editable model | Common representation across tools | Multiple views edit one model | Compatible representations and conflict policy |

These forms can coexist in one workspace. A musical plugin's unusual interface might remain an interactive portal, while its exposed gain parameter becomes a native slider. A PDF renderer might be an ordinary component, while the editor beside it is the user's actual Neovim process. The source application's complex behavior is preserved where that is valuable; new interface code is written only for the composition and the parts deliberately being reshaped.

A pixel crop cannot independently scroll a second copy of the same source viewport. Both crops initially point to one state. That is not a reason to abandon the feature: obtain a second source view, create an independent view through a runtime adapter, or render a new view of an exported model. A virtual display alone does not manufacture a second viewport state.

Likewise, placing a calendar and an issue tracker beside one another does not make a date field mean the same thing in both. A link must specify whether it copies a due date, schedules a work block, follows the selected issue, or merely shows a related event. The composition becomes powerful when those relationships are explicit and manipulable.

## The source code barrier is much lower than it appears

The largest mistake would be to assume that a missing public API leaves only screenshots. There is a substantial middle layer between official integration and reverse engineering machine instructions.

### Discover the stack after encountering the application

A local adapter discovery system can inspect the capabilities it is permitted to see: process and module identity, browser/runtime type, accessibility support, document formats, scripting dictionaries and framework metadata. It then selects an appropriate inspection route. Users should not have to know whether an app is Qt, WPF, AppKit, Electron or a browser canvas.

Several existing tools demonstrate the ingredients. GammaRay inspects running Qt object trees, properties, slots, signals and their connections. Its probe selection accounts for the target's Qt ABI. Snoop exposes WPF trees and properties. UWPSpy inspects and changes running UWP and WinUI elements. These are powerful examples of recovering useful live structure without the original application source. They are inspection/modification tools, not complete cross-app composition products. [GammaRay](https://github.com/KDAB/GammaRay) · [Probe selection](https://docs.kdab.com/gammaray-manual/latest/gammaray-launcher-gui.html) · [Snoop](https://github.com/snoopwpf/snoopwpf) · [UWPSpy](https://github.com/m417z/UWPSpy)

The architectural implication is important: a generic Qt adapter could expose recurring kinds of objects and behavior across many programs, with much smaller app-specific mappings layered above it. The system need not start each integration from zero.

### Export behavior from the running process

Frida provides function interception, native-function calls and RPC exports. Detours provides binary function interception on Windows. DynamoRIO provides dynamic instrumentation and instruction-stream transformation. These establish mechanisms for exposing existing functionality without possessing its source. They do not independently explain an application's business or creative semantics. [Frida API](https://frida.re/docs/javascript-api/) · [Microsoft Detours](https://github.com/microsoft/detours/wiki) · [DynamoRIO](https://dynamorio.org/)

For example, an adapter could discover that a particular interaction invokes a command on a selected object, then expose a typed version of that command to the composition host. It should call the application's ordinary mutation path on the correct thread, preserving validation and side effects, rather than editing arbitrary memory because the screen happens to look right afterward.

Windhawk is relevant as an ecosystem precedent: targeted compiled mods, hooks, symbols and version-sensitive maintenance are already packaged for users on Windows. A composition ecosystem could apply similar distribution machinery to reviewed adapters that expose narrowly scoped objects and actions. [Windhawk development](https://github.com/ramensoftware/windhawk/wiki/Creating-a-new-mod) · [Version and symbol handling](https://github.com/ramensoftware/windhawk/wiki/Development-tips)

### The agent builds a bridge rather than pretending to be the bridge

I would use agents for discovery, experiments, adapter generation, tests and repair. Once a bridge works, ordinary interaction should run through deterministic code. Asking a model to interpret a screenshot for every keystroke would add latency and uncertainty to a task that should behave like software.

A credible pipeline is: identify an observation/action channel; record representative interactions; infer candidate objects and operations; generate an adapter; compare its effects against the original app; test interruption and concurrent edits; then enable a bounded set of writes. When a target changes, rediscover and retest. A confidently generated selector is not sufficient evidence that a destructive action still targets the intended object.

With generous engineering, this can be open-ended rather than confined to a fixed list of apps. It still produces versioned knowledge about each target. “Bring an unfamiliar app and the system learns how to adapt it” is a different and more credible promise than “all software secretly implements one universal interface.”

## Browser applications

The browser is the most promising starting point for broad coverage because it already supplies an instrumentable runtime, rendering tree, network stack and isolation model. There are several distinct approaches.

### Keep the original website running and project its surface

An owned Chromium or Electron browser context can load the original site and display its live surface inside a composed workspace. Offscreen rendering can supply frames or shared GPU textures; the host maps input back to the source. Separate WebContents preserve origin boundaries and can use explicitly chosen session partitions; separate views alone do not isolate cookies or accounts. This preserves the site's real runtime and avoids pretending copied markup is an application. [Electron WebContentsView](https://www.electronjs.org/docs/latest/api/web-contents-view) · [Session partition settings](https://www.electronjs.org/docs/latest/api/structures/web-preferences) · [Offscreen rendering](https://www.electronjs.org/docs/latest/tutorial/offscreen-rendering)

An external browser connection is another option where supported and authorized. The Chrome DevTools Protocol exposes DOM, runtime and accessibility capabilities. However, debug access is a powerful permission, not a harmless view-only connection. Chrome restricts old default-profile debugging patterns and now documents an explicit consent flow for connecting to an existing browser session. Do not design around silent access to every logged-in browser. [CDP](https://chromedevtools.github.io/devtools-protocol/) · [Debugging security changes](https://developer.chrome.com/blog/remote-debugging-port) · [Active-session connection](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session)

### Modify the original document or expose its controls

An extension can inspect and modify the DOM while keeping the application in its original document. Its default isolated world does not automatically expose page JavaScript variables. Main-world instrumentation gives different access and different interference risks. Moving a live element within its document can preserve much more behavior than cloning it, but frameworks may restore its previous structure or depend on ancestor context. A robust adapter needs to understand those dependencies. [Chrome content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts)

Copying a React component's resulting DOM to another page does not move its closure state, event delegation, reconciliation ownership or service session. Canvas-rendered controls may not exist as DOM elements at all. Shadow DOM and cross-origin frames introduce additional boundaries. These are reasons to retain the original runtime or generate a bound proxy, not reasons that websites require source access to be useful ingredients.

### Proxy and virtualize the web application

Webfuse documents a proxy and JavaScript virtualization approach to augmenting live websites, together with extension and automation APIs. That provides another way to mediate behavior without a website's source. BrowserBox's Hyper-Frame approaches embedding through a remote browser. Neither should be understood as an ordinary iframe that somehow makes every origin rule disappear. Proxies and remote browsers move trust and execution boundaries and can alter compatibility with authentication, streaming, uploads or embedded services. Broad vendor compatibility claims need testing. [Webfuse architecture](https://www.webfuse.com/blog/web-augmentation) · [Webfuse automation](https://dev.webfuse.com/automation-api/) · [Hyper-Frame](https://browserbox.github.io/hyper-frame/)

The production design should retain normal isolation and put approved connections through a broker. Disabling browser security for an entire workspace is not an acceptable universal solution. A remote page must never gain native filesystem access merely because it sits beside a trusted local editor.

### Important implementation details

Recent browser primitives strengthen the idea without solving every layer. The same-document `moveBefore()` operation preserves state that ordinary removal and reinsertion can disturb, including focus, iframe loading and certain popup states. It does not move a component across documents or guarantee that its framework accepts a new ancestry. Experimental HTML-in-Canvas explores DOM rendering into graphics surfaces with related geometry and interaction support, but has deliberate cross-origin and security exclusions. These are useful building blocks and browser-fork directions, not universal import APIs. [State-preserving moves](https://developer.chrome.com/blog/movebefore-api) · [HTML-in-Canvas proposal](https://github.com/WICG/html-in-canvas)

Some easily missed host constraints matter. Electron permits a WebContents in only one WebContentsView at a time; multiple projections of the same runtime require a shared surface or deeper compositing. Its input-dispatch API also has a window-focus requirement. CEF's current render-handler source exposes accelerated paint interfaces across Windows, macOS and Linux, despite older prose documentation suggesting more limited support. Source/version checks matter when selecting the implementation. [WebContents input](https://www.electronjs.org/docs/latest/api/web-contents#contentssendinputeventinputevent) · [CEF current render interface](https://github.com/chromiumembedded/cef/blob/master/include/cef_render_handler.h)

## Native desktop applications

### Mac first is realistic

For a macOS implementation, I would build a directly distributed, signed and notarized local application with a carefully scoped native broker. The important public primitives are ScreenCaptureKit for live rendering, Accessibility for exposed controls and state, and event or application-specific dispatch for interaction. A native GPU compositor can arrange the captured surfaces without encoding every frame as video. Apple supplies IOSurface-backed frames and relevant geometry metadata. [ScreenCaptureKit sample](https://developer.apple.com/documentation/screencapturekit/capturing-screen-content-in-macos)

A concrete trap: ScreenCaptureKit's source rectangle is ignored for single-window capture. Capture that window and crop the texture in the host. Capturing a source that is covered is also a different question from forcing a minimized application to keep rendering; target applications and OS versions require separate lifecycle tests. [Source rectangle documentation](https://developer.apple.com/documentation/screencapturekit/scstreamconfiguration/sourcerect)

Accessibility can expose actions, values and notifications. It can be enough to construct a new frontend for some operations; incomplete custom widgets may expose very little. Apple explicitly allows notification registration to fail when unsupported. A system should negotiate real capabilities, then add framework or app-specific access where permitted. [AX notifications](https://developer.apple.com/documentation/applicationservices/1462089-axobserveraddnotification) · [Concrete AX implementation](https://www.hammerspoon.org/docs/hs.axuielement.html)

Mac does not offer a documented general equivalent of taking an arbitrary foreign NSView and making it a normal child of your application. That is an observation about public APIs, not a proof that deeper adaptation is impossible. Keeping the source process and using a surface/action proxy often produces the desired interaction without physically relocating its objects. Where permissible, an in-process adapter can provide stronger framework-level dispatch and export additional state.

This generic broker is unlikely to fit ordinary Mac App Store sandboxing. Apple's documentation identifies assistive Accessibility clients and arbitrary Apple-event sending as sandbox-incompatible; that differs from making one's own app accessible. Capture, Accessibility and automation permissions are separate. Direct distribution removes the store-route assumption, but does not remove system permission or target-process protections. [App Sandbox boundaries](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox) · [Apple developer clarification](https://developer.apple.com/forums/thread/789663)

### Other operating systems have useful advantages

| Platform | Strong route | Important boundary |
|---|---|---|
| macOS | ScreenCaptureKit plus AX and native/runtime adapters | No public general foreign-view transplant; permissions and app focus need orchestration |
| Windows | HWND embedding, DWM surfaces, UI Automation, framework probes | Many modern widgets are not separate HWNDs; integrity levels and hierarchy assumptions remain |
| Linux X11 | Window reparenting and permissive compositor integration | Internal application semantics still require adapters |
| Linux Wayland | Implement at compositor level or use authorized remote-desktop portals | An ordinary client cannot arbitrarily access other clients' surfaces |

Windows SetParent explicitly supports cross-process window relationships, although DPI, styles, messaging and ownership need attention. DWM thumbnails provide live replicas, while UI Automation offers semantic control patterns. Those are complementary mechanisms. X11 reparenting is another real route. Wayland deliberately centralizes surface and input authority in the compositor; a custom compositor is therefore a principled ambitious architecture rather than a blocked ordinary app trying to impersonate one. [SetParent](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setparent) · [DWM thumbnails](https://learn.microsoft.com/en-us/windows/win32/dwm/thumbnail-ovw) · [UI Automation](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-controlpatternsoverview) · [Xlib](https://www.x.org/releases/X11R7.5/doc/libX11/libX11.html) · [Wayland architecture](https://wayland.freedesktop.org/docs/book/Architecture.html)

I would study the compositor-native approach seriously without requiring a Linux migration to prove the concept. Mac-first should retain local apps, with an optional stronger host for software that can run within it. Remote application sessions can improve isolation and input ownership, but they introduce separate installations, logins, licensing and document continuity. They are an architectural option, not a transparent clone of an already-running local session.

### What makes an embedded piece feel real

The missing subsystem is often input and transient UI, rather than rendering. A composed control must support keyboard shortcuts, pointer capture, drag operations, scroll, selection, clipboard, input methods, menus, tooltips and modal dialogs. A dropdown rendered outside the selected crop still belongs to that component. An open-file dialog must identify which real application and document it affects.

One human keyboard focus is perfectly adequate for ordinary composition. Requiring independent simultaneous keyboards would unnecessarily raise the bar. Nevertheless, the host must reconcile its logical focus with the original app's key window and responder chain. Being able to post an event to a process does not prove every application will accept rich background interaction. Apple's targeted keyboard API is a useful primitive, not the whole contract. [Targeted keyboard events](https://developer.apple.com/documentation/applicationservices/1462057-axuielementpostkeyboardevent)

The host also needs its own accessibility representation. A visual portal with no meaningful accessibility tree could make software less usable for people who rely on assistive technology. Where semantic export exists, mirror appropriate roles and actions; elsewhere preserve an obvious path to the original accessible application. Do not claim full accessibility merely because the pixels are correct.

## The closest precedents

The basic idea has a much deeper history than today's agent workspaces. These are the most relevant systems I found, grouped by what they actually contribute. None was independently executed during this research.

### Direct composition of existing interfaces

**WinCuts, 2004.** Microsoft Research implemented live, interactive regions of existing windows as independent windows. It directly demonstrates the spatial “take this part and keep using it over here” interaction. Its remote-sharing extension was read-only, an important distinction from the local interactive version. It does not supply a universal domain model. [Paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2004/01/WinCuts.pdf)

**User Interface Façades and Metisse, 2006.** This is the closest historical expression of the proposed interface manipulation: adapt and recombine existing GUI regions, redirect interaction, and use accessibility information where available. The implementation used a specialized Linux/X window system. It is evidence that the technique was built, not a maintained modern Mac product recommendation. [Project](https://ws.iat.sfu.ca/facades/) · [Paper](https://direction.bordeaux.inria.fr/~roussel/publications/2006-UIST-uifacades.pdf)

**Prefab, 2010 onward.** A toolkit for recovering interface structure from pixels and modifying existing interfaces without source cooperation. This matters because even opaque pixels can support a useful inferred structure, although inference should not be confused with authoritative hidden state. Its reusable annotations and correction mechanisms are especially relevant to generated adapters. [Project](https://prefab.github.io/) · [Toolkit](https://github.com/prefab/code)

**Fusion, 2018.** A direct web precedent for extracting live widgets from existing sites and wiring them into mashups. Its paper reports seven case studies with small amounts of glue code. The prototype's browser/proxy security accommodations and medium-fidelity scope are essential qualifications; it is not evidence that all modern authenticated sites can be securely imported unchanged. [Author-hosted paper](https://pg.ucsd.edu/publications/Fusion-opportunistic-web-prototyping-UI-mashups_UIST-2018.pdf)

**Arcan, Durden and Pipeworld.** This is the most interesting compositor-first lineage to study. Durden documents interactive window slicing; Pipeworld combines hosted applications and streams with typed, spreadsheet-like dataflow. Its public source is experimental and depends on its ecosystem. It offers architectural inspiration beyond a grid of app windows, while external domain semantics still need bridges. [Window slicing](https://arcan-fe.com/2017/09/22/arcan-0-5-3-durden-0-3/) · [Pipeworld](https://arcan-fe.com/2021/04/12/introducing-pipeworld/) · [Source](https://github.com/letoram/pipeworld)

### Strong composition when applications participate

**OLE and COM compound documents.** Existing technology already supports objects and in-place editing supplied by other applications. This proves the value of retaining specialist engines. Its participation interfaces also explain why it does not automatically decompose every unrelated application. [Microsoft compound documents](https://learn.microsoft.com/en-us/windows/win32/com/compound-documents)

**OpenDoc and KDE KParts.** Both offer useful component-oriented lineage: editors and viewers can be parts rather than whole applications. Their lesson is how rich composition can be with a contract; their limitation for this proposal is that legacy software does not automatically expose that contract. [Historical OpenDoc explanation](https://preserve.mactech.com/articles/develop/issue_22/opendoc.html) · [KParts](https://techbase.kde.org/Development/Tutorials/Using_KParts)

**Webstrates, Codestrates and Varv.** These support malleable behavior and composition within a deliberately designed substrate. They are good references for the new host's own programmable layer, but an arbitrary SaaS app does not become a participating object merely because it runs in a browser. [Publications](https://webstrates.net/project/publications/) · [Varv](https://vis.mit.edu/pubs/varv/)

**Engraft.** Rich programming tools can recursively embed other tools and computations. This is relevant to making the user's glue editable and inspectable, rather than burying everything in generated scripts. It is a component contract, not an extractor of arbitrary proprietary app internals. [Project and paper](https://engraft.dev/)

### Extract and connect existing information

**Chickenfoot, d.mix, WebMakeup, Wildcard and TabFS.** These explore in-browser automation, sampling visible content into service calls, persistent page augmentation, spreadsheet-like customization, and exposing browser state through a filesystem. Their specific mechanisms vary; none alone is the proposed universal desktop. Together they show that users can point at something meaningful in an existing tool and acquire a reusable operation without first learning its entire stack. [Chickenfoot](https://publications.csail.mit.edu/abstracts/abstracts06/rcm/rcm.html) · [d.mix](https://hci.stanford.edu/research/mashups/) · [WebMakeup thesis](https://ekoizpen-zientifikoa.ehu.eus/documentos/5ecb7f7c2999521315203e3b) · [Wildcard](https://github.com/geoffreylitt/wildcard) · [TabFS](https://omar.website/tabfs/)

**Cambria.** Bidirectional schema lenses are a particularly useful model for connecting different representations while preserving source-native information. The research also demonstrates unavoidable tradeoffs when schemas express different things. This is a better foundation than assuming one generic JSON schema can faithfully represent every tool. [Cambria](https://www.inkandswitch.com/cambria/)

**Synchronising Content Across Formats In the Wild, 2025.** This research explicitly works with an existing heterogeneous tool ecology, including publishing tools and format conversions, instead of assuming everyone will adopt a replacement substrate. It is unusually close to the intent behind your proposal. Its distinction between practical conversion and full bidirectional synchronization is worth preserving. [Paper](https://software-substrates.github.io/proceedings/2025/statements/Paper%2010%20-%20Yann%20Trividic%20-%20Synchronising%20Content%20Across%20Formats%20In-the-wild.pdf)

The broader malleable-software literature identifies both shared data and a shared interaction environment as necessary. Your proposal contributes a particularly consequential constraint: retain the actual heterogeneous tools and learned practices people have, rather than making a new substrate the price of admission. [Ink and Switch synthesis](https://www.inkandswitch.com/essay/malleable-software/)

## What current products do and do not already solve

| System or category | Useful existing contribution | Gap relative to your proposed app |
|---|---|---|
| OnTopReplica | Live Windows window regions and click forwarding | Domain state and robust focus/lifecycle composition |
| Webfuse | Proxy-based live web augmentation and automation | Arbitrary native apps and independently verified universal compatibility |
| BrowserBox and Hyper-Frame | Remote browser sessions presented in another surface | Native app composition and the semantic relationship layer |
| GammaRay, Snoop and UWPSpy | Framework-aware inspection and modification | End-user composition, security broker and multi-app semantics |
| Windhawk and Frida | Runtime adaptation and function hooks | Generated, validated component contracts and user-facing host |
| Microsoft UFO and Mediar | Hybrid UI/API automation and target discovery | Persistent, directly manipulable composed application surfaces |
| Screenpipe | Observation/history and workflow context | Extraction of original live behavior as composable components |
| MCP Apps | Authored interactive UIs and host-mediated tools | Automatic extraction of existing app interfaces |
| Agent workspace plugin ecosystems | Convenient homes for newly authored surfaces | Adapting arbitrary pre-existing tools without rewriting their functionality |

The distinctions matter when choosing dependencies. Current BrowserBox describes its core as proprietary/binary-distributed; historical open-source descriptions are not reliable for the current product. Rewind is historical rather than an active capture foundation following its announced capture sunset. A product's ability to record a screen or run an agent against it is not evidence that it can supply continuously interactive components. [BrowserBox current repository](https://github.com/BrowserBox/BrowserBox) · [Limitless announcement](https://www.limitless.ai/) · [Screenpipe](https://github.com/screenpipe/screenpipe) · [OnTopReplica](https://github.com/LorenzCK/OnTopReplica)

MCP Apps is valuable for presenting adapter-authored controls through a defined host boundary. It does not discover those controls inside an arbitrary application. Similarly, an existing agent shell might host the finished composer or expose its tools, but the core research problem sits below its plugin interface. [Unpublished prior-research reference omitted.] [MCP Apps overview](https://apps.extensions.modelcontextprotocol.io/api/documents/overview.html) · [UFO hybrid actions](https://github.com/microsoft/UFO/blob/main/documents/docs/ufo2/core_features/hybrid_actions.md) · [Mediar](https://github.com/mediar-ai/terminator)

## The architecture I would build

The following is a proposed system, not an existing implementation. Its central abstraction is an **app part with a contract**, backed by an original runtime. A part can be rich without becoming independent of that runtime, just as a network-backed control can be a real part of an ordinary app.

### Runtime and session ownership

A runtime manager either attaches to an explicitly authorized existing instance or starts a dedicated instance. It knows the process or browser target, account, document, supported capabilities and lifecycle. Attaching and launching are different modes: launching a second copy does not preserve the first copy's unsaved work or current session by magic.

The manager retains the original engine, its specialist plugins and its backend relationship. It controls staging, capture scheduling and reconnection. It records whether a source must be visible, may be covered, or can render offscreen. App-dependent compromises are visible, rather than silently presenting an old frame as current.

### Component export

Each exported part declares its identity, source owner, view representation, sizing rules, supported operations, change stream, focus behavior, popup ownership, accessibility and disconnect behavior. A source-side adapter may export an existing view, create another view bound to the same model, or publish data and commands that a new host-side view consumes.

This gives the system several ways to preserve functionality. There is no requirement to force an original native widget into a foreign process. Its model and command handlers can remain where they work correctly while the host presents a live view or a deliberately redesigned control.

### Three separate execution paths

The rendering path carries textures, display updates or semantic view data. The interaction path carries focus, gestures, keyboard input and transient UI. The control path carries typed reads, operations and subscriptions. These paths can share a transport but must not be confused. A fast GPU path cannot explain a record's identity; a correct record API cannot render a sophisticated canvas by itself.

For macOS, I would combine a Swift/AppKit integration layer and native graphics composition with an isolated systems-language broker. A controlled Chromium/CEF runtime would supply the strongest web-hosting path; an authorized extension or debugger bridge would preserve existing browser sessions when appropriate. Exact language choices are less consequential than the separation of trust, lifecycle and responsibility.

A browser fork becomes reasonable if the product needs stronger element surfaces, retained offscreen rendering, precise hit testing and multi-projection behavior than ordinary embedding exposes. That is within your generous engineering premise. It still should preserve origin isolation and place cross-app access behind explicit capabilities.

### Relationship objects

The lasting user-created object should be more than a saved layout. It should capture a relationship such as:

- Follow the document I am editing and show the corresponding compiled preview
- Use the item selected in this source as the query in that tool
- Expose these three parameters from this device, with their original units
- Pin this source excerpt and its citation to this project
- When this calculation changes, update this supported destination field

Each relationship records the source identities, transformation, direction, trigger, authority, freshness and failure behavior. It can be inspected and modified without rebuilding the whole workspace. The layout is one way of presenting those relationships.

This is where the concept becomes distinctive. A user should be able to acquire a useful behavior while working, then reshape how it participates in the rest of their environment. They should not need to first become an integration engineer or replace their tools with a universal low-fidelity data model.

### Durable recipes and verified repair

Store compositions as portable local recipes: source references, layouts, adapters, relationships, permissions and tests. Avoid storing raw session credentials inside shareable recipes. A person can share the shape of a workflow while each recipient supplies their own authorized applications and accounts.

On restart, the host reopens or reacquires sources and checks identities. A window titled “Untitled” is a locator, not durable identity. If an adapter loses its target after an update, the system can propose or validate a repair; it should not silently bind to a similarly named but different object.

Versioned adapters, example interactions and regression fixtures become the practical ecosystem. They make successful adaptation cumulative. This is also where the product could develop a durable advantage: reliable integrations and their behavioral knowledge, rather than a particular model or an attractive container UI.

## Preserving meaning and live state

The most important architectural choice is to let original applications continue owning the things they understand. The composition host should own arrangement, relationships and provenance. It can cache source state, but a cache must not silently become a rival source of truth.

Separate durable state, live unsaved state, ephemeral interaction state and derived output. An editor's buffer is not necessarily its file on disk. A compiled PDF is not necessarily current with that buffer. A selected track is not the same thing as an entire music project. A field displayed optimistically in a browser may not yet be committed on the server.

Where existing interfaces expose those distinctions, use them. For example, editor APIs can expose dirty buffers, document versions and selections; file watching alone cannot substitute for that. LSP document synchronization is useful within a participating editor/server relationship but does not automatically attach to any arbitrary editor. [VS Code document API](https://code.visualstudio.com/api/references/vscode-api) · [LSP document synchronization](https://github.com/microsoft/language-server-protocol/blob/gh-pages/_specifications/lsp/3.18/specification.md)

### Identity before synchronization

A useful object reference includes application, account/workspace, document, source object and version or session identity. Paths, labels and screen coordinates are insufficient on their own. Renaming should not break a relationship; a duplicated label should not redirect one.

Use events with source identity and origin markers so an update sent from A to B does not endlessly bounce back from B to A. Distinguish “command accepted” from “effect observed” and “durably committed.” After a timeout, some operations are genuinely uncertain. Blindly retrying can duplicate an import, submission or other external effect.

### Different representations need declared mappings

A normalized plugin value and a value in decibels may describe the same parameter differently. A rich document and Markdown may not carry the same annotations. A compiled formula does not uniquely identify the macro that generated it. These differences are manageable when their mappings and losses are explicit.

CRDTs are useful for shared models and concurrent edits, but do not automatically make an opaque application a participant. Automerge can preserve conflicting values while selecting a deterministic visible winner; that is not a decision about which creative intention should win. Yjs provides origin-scoped undo and stable relative positions inside its model; an adapter still needs to map those to the source application's representation. [Automerge conflicts](https://automerge.org/docs/reference/documents/conflicts/) · [Yjs undo](https://docs.yjs.dev/api/undo-manager) · [Yjs relative positions](https://docs.yjs.dev/api/relative-positions)

### Transactions should match the promises actually made

The host can offer strong transactions for its own layout and relationship state. Across independent sources, it can checkpoint, apply, verify and compensate where possible. Full all-or-nothing behavior requires stronger cooperation from the participants. This is a limit on a particular guarantee, not on composition itself.

Google Docs illustrates useful scoped guarantees: its batch API supports revision-aware writes and atomic application of a batch. Those guarantees do not automatically include a second app. Its current comments/suggestions API also documents partial persistence outcomes that a bridge should inspect. A real contract is more informative than simply marking an integration “read/write.” [Docs batch operations](https://developers.google.com/workspace/docs/api/reference/rest/v1/documents/batchUpdate) · [Comments and suggestions](https://developers.google.com/workspace/docs/api/how-tos/suggestions)

## How this could feel as a product

I would design around acquiring and connecting parts in the course of using one's existing tools. The user points at a region, object or action, asks to keep it available elsewhere, and the system discovers the richest viable connection. It shows what was acquired: a live view, a source-bound control, an identified object, or a read-only excerpt. Those distinctions should be available without covering the workspace in technical labels.

The next interaction is relational: “have this follow my selection,” “keep these values linked,” or “show this preview beside what I am editing.” The system can propose a sensible mapping and explain its consequential effects. Ordinary behavior then runs without an agent narrating every update.

An expanded view should always recover the original application's full context. A person should not get trapped in a tiny extracted panel when they need an uncommon operation. Transient UI can expand a part temporarily or appear as its attached surface. The system should preserve familiar shortcuts and meaningful spatial arrangements rather than continually rearrange the workspace based on guessed intent.

A proposed version would preserve an existing editor and its interaction habits while bringing relevant tools to it. The goal is not to turn every surface into one uniform visual style. The unusual affordances of the original tools may be exactly what you want to retain.

### A study and LaTeX environment

Keep the real editor process, configuration, mappings and unsaved buffer. Attach a PDF preview and browser research surface. A selected source location drives the preview when the mapping is known; a browser excerpt can be inserted with provenance through a verified editor command.

Neovim is an especially useful baseline because its remote UI protocol allows external clients, per-window grids and externalized interface elements. That reuses the editor engine rather than simulating Vim in a new text widget. Traditional Vim needs its own appropriate integration path; the product should identify the actual installed tool. [Neovim UI protocol](https://neovim.io/doc/user/api-ui-events/)

Compile an immutable source revision and associate the PDF and synchronization data with it. If the user continues typing, the preview can remain useful while clearly belonging to an earlier revision. SyncTEX can link source locations and rendered geometry; it is not a general inverse that converts any visual PDF edit into the intended LaTeX change. [SyncTEX implementation description](https://mirrors.ctan.org/info/knuth-pdf/xetex/xetex-changes.pdf)

This is a good test of the host's quality. Because Neovim already offers a rich interface, it is not by itself a test of adapting an unfamiliar closed app. That requires the second experiment described below.

### A music environment

Keep Ableton and the actual plugin running. Pin the plugin's original editing surface, expose selected supported parameters as custom controls, and connect a browser or note surface to the project and musical location. An agent-created musical tool could participate without having to rebuild the DAW or the plugin.

Live's object interfaces are meaningful but bounded: observability and mutation have threading and sequencing constraints. Availability depends on the application's edition, installed components and exposed interfaces; Max for Live availability is not assumed. Existing MIDI/control/plugin interfaces or an authorized adapter would need to be selected accordingly. [Live observer interface](https://docs.cycling74.com/reference/live.observer)

Real-time audio should stay on the existing audio engine's scheduling path. A control broker or LLM loop should not become part of sample-accurate DSP. Current Ableton documentation includes Link Audio, so older statements that Link never carries audio are obsolete; nevertheless, audio or tempo exchange is not shared clip, device or project state. [Ableton synchronization and Link Audio](https://www.ableton.com/en/manual/synchronizing-with-link-tempo-follower-and-midi/)

### A genuinely unsupported native app

Select a useful inspector or specialized control in an app with no prewritten adapter for that feature. Discover its framework, observe a normal interaction, expose a property/action and its change signal, then bind a new control to the same source object. Reuse that binding in another composition without recording the interaction again.

This is the diagnostic example for the ambitious thesis. It tests whether the system can acquire capabilities from unfamiliar running software, rather than merely offer another collection of integrations somebody had already built.

## Boundaries that remain under the generous engineering premise

I am not treating “too complicated,” “requires reverse engineering,” or “will need ongoing maintenance” as refutations. Those are engineering scope. The following boundaries are different.

**Observation is bounded by permitted access.** A blank accessibility tree does not prove an app is unknowable: a framework probe or runtime trace may reveal much more. But if two states are indistinguishable through every channel available and authorized to the implementation, it cannot guarantee which state exists. A client also cannot recover server-only records that never reach it.

**An action needs authority.** Rearranging a website does not give its logged-in account a new server permission. Local control does not override an enterprise policy or protected process. Authentication and security prompts need their proper origin and context. A passkey's relying-party rules are not erased by moving a login surface to a proxy origin. [WebAuthn specification](https://www.w3.org/TR/webauthn-3/)

**Some mappings lose information.** A rendered picture, flattened document or audio waveform cannot uniquely recover its original editing model. The system can retain that model, expose it through a bridge, or offer an explicitly approximate reconstruction. It cannot promise a unique inverse where none exists.

**Independent services retain independent outcomes.** A sent message or an externally committed action may not be reversible. A useful composed workspace can still exist, but must not promise universal rollback.

**Runtime protections affect available routes.** Some instrumentation is blocked by code signing, hardened runtime or process protections. A production system should use another permitted channel or declare the affected feature unsupported. Requiring people to weaken core OS security would materially change the product, and is not the foundation I recommend. [Apple library validation](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.cs.disable-library-validation) · [Windows process access](https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights)

**Service and distribution policies are separate from technical rendering.** An embedded browser may render a service yet be unacceptable for its login flow. Google's OAuth policies are a concrete example. Application terms, redistribution rights and implementation-specific legal questions need review before shipping; this is not a claim that all user-side modification is prohibited or permitted. [Google OAuth policies](https://developers.google.com/identity/protocols/oauth2/policies)

## Trust is part of the composition model

This app would be unusually powerful because it sits between programs the user trusts separately. A malicious page must not become able to control a local app just because both are placed in the same workspace. A generated adapter must not receive every source's session, files and action privileges by default.

I would use separate origin/process boundaries; authenticated local channels; source-specific read/action permissions; a permission inspector; visible source/account identity; and revocable relationships. A recipe may contain an instruction such as “read this field from this account and send this value to that tool,” with the relevant authorization. It must not smuggle broad privileges through a convenient drag-and-drop gesture.

Treat content inside apps as data, not instructions for the host. An email saying “connect the payroll folder” must not authorize a new integration. The same applies to websites, retrieved documents and agent-generated adapter suggestions. In the browser host, keep remote content sandboxed, context-isolated and away from unrestricted native IPC. [Electron security recommendations](https://www.electronjs.org/docs/latest/tutorial/security)

Permission should be granular at the semantic level wherever possible. “Observe this document and invoke these three operations” is better than a component simply inheriting the broker's entire ability to control the desktop. The underlying OS grant may be broad; the product still needs narrower internal enforcement.

## What would convincingly establish viability

A polished video of several clipped windows would establish too little. A prototype using only cooperative APIs would also leave the source-independent claim unanswered. I would run two parallel tracks.

### Track one validates the host

Use well-understood interfaces, including a real editor process, browser app and native component, to prove that the composed experience is coherent. Test typing, selection, shortcuts, IME, menus, drag/drop, undo, accessibility and transient windows. Keep originals and composed views open together. Changes must affect the same underlying objects, not disconnected replicas that happen to look similar.

Measure visual and semantic latency separately. A suggested local target such as p95 below 100 milliseconds for ordinary observed property changes is a product target, not a result of this research. Compilation, remote services and audio scheduling need their own measures. “Real time” should not conceal seconds of polling in one adapter and low-latency subscription in another.

### Track two validates discovery

Choose unfamiliar applications and features before creating adapters. Include a native framework app, a custom-rendered app, a complex authenticated website and a site with virtualized content. Record how much manual assistance is needed, what was discovered, what remained opaque and whether the resulting bindings are reusable.

The decisive demonstration is an unsupported source feature becoming a verified component: source edits appear in the host; host edits invoke the correct original behavior; the same binding works in a second composition; a reload or target replacement either reconnects correctly or stops clearly. Generating plausible adapter code is not the outcome being tested.

### Stress the parts that impressive demos omit

| Test | What success establishes |
|---|---|
| Duplicate document names and renamed objects | Identity is not just a label match |
| Source rerender, resized window and changed scale | Input and selection follow the intended target |
| Covered, minimized, hidden and crashed sources | Liveness and recovery are explicit |
| Two slices of one source | Shared or independent viewport behavior matches the promise |
| Unsaved edits and slow preview build | Derived output carries the correct revision |
| Concurrent source and host edits | Conflicts or ownership policy prevent silent overwrite |
| A cyclic value connection | Updates quiesce rather than oscillate |
| Timeout after an accepted command | Retry does not blindly duplicate an effect |
| Menus, IME and native dialogs | Composition preserves actual interaction rather than clicks alone |
| Permission revocation and malicious source text | The host maintains boundaries between applications |
| A new application version | Repairs are verified and false successes are counted |

The evaluation should distinguish visual fidelity, behavioral fidelity, semantic fidelity and continuity across changes. A live original canvas can excel at the first two with limited semantic export. An API-backed record can have excellent identity while omitting the original rich editor. Reporting those separately makes the capability envelope useful rather than evasive.

## Product viability and where I would start

The concept is strongest for people whose valuable work already spans several specialized tools and who repeatedly invent their own ways of connecting them. The benefit is keeping their existing expertise while making the environment adapt around it. That is a more specific proposition than another place to open tabs.

My product hypothesis is that people would value durable workflow fragments: an actual editor plus its preview and references; a specialist inspector plus controls from another tool; a research object whose identity follows across applications. This research does not establish market size, willingness to pay or sustained adoption. Those need observation of real use, including whether the composer saves attention after the novelty wears off.

I would start with a Mac application that combines native sources and browser sources, and deliberately supports both an excellent known-interface path and one strong source-independent discovery path. A browser-only prototype would be informative, but would not settle the desktop claim. An API-only prototype would be useful, but would not settle the adaptation claim. Both tracks should exist from the start.

The core worth building is the runtime manager, component contract, interaction broker, relationship graph and adapter verification system. Reuse graphics/browser infrastructure, existing editors, original application engines and established integration mechanisms. Do not rebuild LaTeX editing, a DAW, a browser engine and a notes database merely to demonstrate a shell around them.

I would study Metisse/UI Façades and Arcan/Pipeworld for composition, Chromium/CEF for web hosting, GammaRay/Windhawk/Frida for runtime adaptation, and Cambria/Engraft for the editable relationship layer. They supply complementary lessons; none is an obvious complete fork-and-rebrand foundation for your specific Mac-first goal.

The strongest long-term version could acquire a tool from an unfamiliar app on demand, make the acquired behavior legible and reusable, and maintain it as that app changes. It would gradually accumulate a library of adapters and relationship patterns while leaving the source tools replaceable. That is an ambitious but coherent engineering program.

My recommendation is to pursue the idea as **composition of running capabilities**, with original tools retaining their engines and a malleable host owning the connections. There is enough concrete precedent to move beyond debating whether the concept is possible. The genuinely open question is how broadly and transparently the discovery and hosting layer can work, and whether the resulting compositions remain useful, trustworthy and easy to reshape in daily use.

## Where to read first

For the closest demonstrated interface, read the [UI Façades paper](https://direction.bordeaux.inria.fr/~roussel/publications/2006-UIST-uifacades.pdf). For the current passthrough analogy, inspect the [SkyCraft protocol](https://github.com/chasmlol/SkyCraft/blob/main/protocol/skycraft_protocol.h) alongside its README. For actual web-widget reuse, read [Fusion](https://pg.ucsd.edu/publications/Fusion-opportunistic-web-prototyping-UI-mashups_UIST-2018.pdf). For a compositor/dataflow direction, explore [Pipeworld](https://github.com/letoram/pipeworld). For source-independent semantic discovery, study [GammaRay](https://github.com/KDAB/GammaRay). For keeping unlike tools' representations intact, read [Cambria](https://www.inkandswitch.com/cambria/) and [the in-the-wild publishing research](https://software-substrates.github.io/proceedings/2025/statements/Paper%2010%20-%20Yann%20Trividic%20-%20Synchronising%20Content%20Across%20Formats%20In-the-wild.pdf).


