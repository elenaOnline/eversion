> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Runtime adaptation and semantic bridges for a composable desktop

Research date: 2026-10-07. Read-only primary-source research. No tools were installed or projects executed. Product capabilities below are documented capabilities, not independent performance validation. Architectural proposals are explicitly identified as inference.

## Bottom line

A source-independent desktop composer is technically much more plausible than a pixels-only answer suggests. Running applications already contain reusable behavior, object graphs, command handlers, model state and renderer boundaries. Existing instrumentation can call or intercept functions; existing framework inspectors can access meaningful live objects; accessibility and DOM offer portable semantic surfaces. Those are ingredients for constructing an adapter that turns part of an existing application into an externally controlled live component.

The important distinction is **no original source required** versus **no knowledge ever required**. Runtime discovery can identify a stack after the fact, then choose a framework adapter or infer an app-specific interface. It need not begin with source or known stack, but it must eventually establish what data and commands mean. Unlimited engineering makes broad coverage credible; it does not make server-only state available locally or abolish OS trust boundaries.

## 1. The game analogy is real and is richer than image overlay

### Actual GTA/Minecraft passthrough projects

The GTA V Enhanced × Minecraft Steve project documents two games running simultaneously: a ScriptHookV plugin supplies GTA camera/collision data, a Fabric mod runs Minecraft, and ReShade depth-composites the Minecraft rendering. Minecraft events become GTA effects: explosions, arrows, movement, and block collision. It is explicitly tied to specific game versions and warns that GTA updates break ASI compatibility. This is not equivalent to a universal importer; it is a purpose-built bilateral adapter. [Project README](https://github.com/utku6767/Gta-V-Enhanced-Minecraft-Steve-Passthrough/blob/main/README.md)

A more explicit implementation, Wither Storm GTA passthrough, documents local WebSocket exchange of camera, player, input, and ground samples; Minecraft exports color, depth, and HUD/hand layers via a Windows shared-memory mapping; a ReShade add-on uploads and composites them. The Forge bridge reports actions such as entity grabs and terrain consumption; GTA implements corresponding forces, removals and effects. Its limitations include non-destructible native terrain/buildings, GPU/CPU cost, and dependency on a usable depth buffer. [Repository](https://github.com/VortexisTV/wither-storm-gta5-passthrough)

The upstream Universal Modder repo explicitly distinguishes passthrough mods from asset ports, decompositions and reimplementations, and includes a Minecraft–GTA example. It uses game recon followed by engine-specific tooling, documented field notes, and a verified working slice; its existence does not establish automatic support for every game. [Upstream](https://github.com/rehan-remade/universal-modder)

Important false-positive: GrandTheftMinecraft is a native GTA mod recreating Minecraft-like behavior with GTA objects and locally sourced Minecraft assets. It is a different architecture from concurrently running Minecraft and synchronizing its engine. [GrandTheftMinecraft](https://github.com/cyteon/GrandTheftMinecraft)

The exact bare project name “MinecraftGTA” was not resolved to a unique primary repository in this search. Use the explicit verified project names above; do not silently conflate them. SkyCraft is covered by the other research stream.

### Desktop translation (inference)

Replace camera/depth/input/gameplay events with viewport/layout/input/document state/actions. Each live piece needs:

- A rendering surface or semantic representation
- An identity and lifetime handle
- Its authoritative state owner
- An action endpoint and event stream
- Coordinate, text-selection and data-type conversion
- A contract for focus, popup windows, modal state, undo, failures and permissions

“An editor from A beside a table from B” is mostly display/input composition. “Dragging a selected entity in A changes a record in B” also requires semantic mapping, transaction rules and verification. The game projects demonstrate exactly why both layers matter.

## 2. Existing runtime instrumentation can expose functionality

### Frida

Frida documents interception/replacement of native functions, calling native functions by address and ABI, runtime-specific bridges, and an RPC export mechanism to a controlling program. Therefore a custom adapter can expose discovered functions to a separate UI and observe relevant calls. This is directly more powerful than forwarding mouse coordinates. Frida does not infer a stable domain API automatically: the integrator still needs correct argument types, object lifetime, thread rules and semantics. It also explicitly reports code-signing policy and whether its interception facilities are available. [JavaScript API](https://frida.re/docs/javascript-api/)

**Inference:** a Frida-based adapter could call “set selected record,” subscribe to an internal “document changed” method, and render a separate control surface, while leaving the actual application engine in place. It should prefer ordinary command handlers and keep mutations on the application's expected execution thread rather than blindly editing memory.

### DynamoRIO

DynamoRIO can transform an unmodified program's running instruction stream, not merely insert tracing callbacks. It supports stock Windows, Linux and Android, with experimental macOS support. Its code manipulation API documents transformations and real restrictions, so it is a strong lower-level fallback rather than a semantic desktop framework. [Project](https://dynamorio.org/), [Code manipulation API](https://dynamorio.org/API_BT.html)

**Inference:** dynamic traces can help discover where UI actions reach state transitions or where a renderer consumes a model, then supply evidence for a higher-level adapter. Observing a machine instruction does not itself explain “which customer is selected.”

### Microsoft Detours

Detours dynamically redirects binary functions to replacement functions, preserving the original implementation through a trampoline. It changes in-memory execution rather than requiring the application's original source. It is a proven interception primitive, not an automatic UI export system. [Microsoft overview](https://github.com/microsoft/detours/wiki)

### Windhawk

Windhawk is an existing Windows mod distribution and lifecycle system. A mod is C++ compiled to a DLL and loaded in each targeted process; its APIs install function hooks. Symbol utilities include binary-version-specific caching and typed wrappers. These are concrete precedents for a per-app adapter ecosystem and update maintenance. [Mod development](https://github.com/ramensoftware/windhawk/wiki/Creating-a-new-mod), [Development tips](https://github.com/ramensoftware/windhawk/wiki/Development-tips), [Project](https://github.com/ramensoftware/windhawk)

**Inference:** the desktop-composition version of Windhawk would distribute signed, versioned semantic adapters, rather than only cosmetic or behavioral patches.

## 3. Framework introspection is the most underappreciated middle layer

These examples show that one need not choose only between public APIs and arbitrary binary reverse engineering.

### GammaRay: Qt

GammaRay can browse live QObject trees, edit properties, invoke slots, monitor signals, and inspect signal/slot connections. It also understands higher-level Qt models, state machines and scene graphs. This is unusually close to exposing application functionality as a graph. [Primary repository](https://github.com/KDAB/GammaRay)

It can attach to an already-running process or launch with a probe. Its launcher identifies Qt versions and matching probes, and refuses unavailable ABI combinations. This supplies a concrete model for runtime stack detection plus compatible adapter selection. [Launcher documentation](https://docs.kdab.com/gammaray-manual/latest/gammaray-launcher-gui.html), [CLI](https://docs.kdab.com/gammaray-manual/latest/gammaray-command-line.html)

### Snoop: WPF

Snoop inspects visual, logical and automation trees in running WPF applications without requiring a debugger; it changes properties, inspects triggers and watches property changes. [Project](https://github.com/snoopwpf/snoopwpf)

### UWPSpy: UWP/WinUI 3

UWPSpy inspects and manipulates running UI elements and properties. It is a framework-specific source-independent view into the actual widget tree. [Project](https://github.com/m417z/UWPSpy)

**Inference across these tools:** build generic framework adapters first, then thin app-specific semantic mappings. A control can be rendered as original pixels, restyled in-place, or represented by a new control bound to the same underlying command/property. These tools demonstrate introspection and modification; they do **not** prove that arbitrary cross-process widget reparenting preserves every behavior. A remote view/controller proxy is often a better abstraction than physically moving an object out of its original process.

## 4. Semantic access without injection

### Accessibility is already a partial capability protocol

Windows UI Automation control patterns expose methods, properties, relationships and events for meaningful behaviors, including Invoke, Value, Selection, Grid and Scroll. One element can implement several patterns. This can drive a substitute frontend without replicating all original screen interactions. [Microsoft control patterns](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-controlpatternsoverview)

On macOS, Hammerspoon's AX wrapper reads and writes attributes and invokes supported actions; observers subscribe to UI changes. Its documentation is candid that applications decide what they expose and may decline changes or report success without the intended effect. [AX module](https://www.hammerspoon.org/docs/hs.axuielement.html), [Observers](https://www.hammerspoon.org/docs/hs.axuielement.observer.html)

**Inference:** AX/UIA are good bootstrap adapters and stable-ish control handles, but are incomplete for virtualized canvases, custom widgets and non-UI domain state. A serious composer should enrich rather than discard them.

### Browser extensions/userscripts

Chrome content scripts can inspect/modify a page DOM and communicate with an extension. Their default JavaScript world is isolated from the site's JavaScript; the MAIN execution world shares the page's environment and consequently its interference risks. These are real mechanisms for modifying a closed-source website's live client UI. [Content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts), [Execution-world manifest setting](https://developer.chrome.com/docs/extensions/reference/manifest/content-scripts)

**Inference:** userscripts/extensions can expose data/actions, observe DOM mutations, add new surfaces and bridge to a desktop host. Copying DOM markup alone does not preserve event handlers, closure state, framework reconciliation or server authorization. Keep the original live instance as authority or deliberately reconstruct a proxy with action bindings.

### Webfuse: live web virtualization/proxy

Webfuse documents an augmented proxy, rewritten requests/DOM, JS sandbox and session extension environment that changes a live site without needing its source. Its automation API produces DOM snapshots, including cross-frame/shadow traversal and stable Webfuse element IDs, and supports actions against those targets. This is an existing vendor architecture adjacent to source-independent web composition; broad compatibility and performance claims are not independently established here. [Architecture](https://www.webfuse.com/blog/web-augmentation), [Automation API](https://dev.webfuse.com/automation-api/)

**Inference:** its relevance is control over the live runtime environment plus identity/action mapping. It is not evidence that any arbitrary authenticated site will remain compatible with all proxy transformations. A proxy also becomes a significant trust/data-handling boundary.

## 5. Adjacent products and research: useful pieces, not the full composition layer

- **Terminator / Mediar:** existing Windows-only automation SDK/MCP with element locators, workflow recording and browser extension support. It combines pixels, DOM and accessibility. The repository claims background interaction, but app-specific behavior must be verified; headline speed/success claims are vendor claims. It provides an actuation layer, not a spatial composer. [Repository](https://github.com/mediar-ai/terminator)
- **Microsoft UFO:** a documented hybrid GUI/API agent architecture mixes UIA with native COM/REST/MCP capabilities and chooses per step. It validates the value of a heterogeneous backend rather than insisting on a single universal API. It is workflow automation rather than extracted live component composition. [Hybrid action layer](https://github.com/microsoft/UFO/blob/main/documents/docs/ufo2/core_features/hybrid_actions.md)
- **Screenpipe:** current repository emphasizes local screen/audio history, workflow mapping and agent context. It is source-available, with separate privacy implications for cloud integrations. It can supply observation/context; its current README does not establish live functionality extraction or arbitrary UI transplantation. [Repository](https://github.com/screenpipe/screenpipe)
- **Rewind:** historical capture/search precedent only. Limitless says the latest Rewind update disables screen/audio capture starting December 19, 2025. Do not recommend it as a currently operating foundation. [Official announcement/FAQ](https://www.limitless.ai/)
- **OnTopReplica:** genuine live window/subregion cloning with DWM thumbnails and click forwarding. This demonstrates useful cropped portals, but issue reports show app-specific click behavior and focus-stealing. It does not expose document semantics. [README](https://github.com/LorenzCK/OnTopReplica/blob/master/README.md), [Focus issue](https://github.com/LorenzCK/OnTopReplica/issues/65)
- **Prefab (Dixon/Fogarty):** direct research precedent for runtime interface modification without source or application cooperation using pixel-level reverse engineering. Its published work includes recovering interface content/hierarchy and advanced interaction behaviors. It establishes the lineage of semantic reconstruction from visuals, not a currently universal production composer. [Project/papers](https://prefab.github.io/papers.html), [University talk](https://engineering.oregonstate.edu/colloquium/1327)

## 6. What unlimited engineering changes, and what it does not

### Engineering scope, not impossibility (inference)

A large adapter catalog can cover recurring frameworks. A recon pipeline can identify loaded modules, inspect accessibility and runtime metadata, trace actions, and generate candidate adapters. Deterministic checks can test state/action correspondence. A runtime can pin versions, select signatures, run regression workflows, rebuild adapters after updates, and degrade to pixels or ordinary controls when semantic confidence drops.

The hard work is obtaining a **correct semantic contract**: which object owns state, which action changes it, what makes a handle stale, whether a callback means requested or committed, and how a multi-app operation can be rolled back or explained if partly complete. This is tractable per integration and can become increasingly automated. It is not solved merely by having a language model recognize a button.

### Genuine boundaries

- **Remote hidden state:** a local hook observes what reaches the client. It cannot reveal server-only records, private computations or unauthorized capabilities absent from the session. The correct model is an authenticated client façade, with the server remaining authoritative. This is an architectural inference, not a claim about a particular service.
- **OS security:** Windows protected processes restrict memory access and thread creation. Apple's hardened runtime normally constrains loaded libraries by signatures/team identity. A production design should respect those boundaries and use sanctioned accessibility, plugins, browser APIs or vendor cooperation when injection is unavailable. No protection bypass is proposed. [Windows process rights](https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights), [Apple library validation](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.cs.disable-library-validation)
- **ABI and lifecycle:** a function address and signature are not portable API guarantees. Compiler/runtime updates, object destruction, UI thread affinity and reentrant calls matter. GammaRay's probe matching is a concrete example of engineering this boundary rather than pretending it disappears.
- **Licensing/distribution:** inspect each chosen dependency and application's terms before packaging. Permission to run software, permission to modify one's running instance, and permission to redistribute code/assets are separate questions. This memo makes no jurisdiction-specific legal conclusion.
- **Safety/privacy:** adapters run close to valuable application state. Prefer narrowly scoped read/action capabilities, local transport with actual authentication, audit trails, explicit irreversible-action gates and no raw credential export. The GTA sample's unauthenticated loopback channel is a prototype warning, not a desktop security design to copy.

## Suggested thesis for the parent answer

The promising idea is a **runtime component adapter system**: discover an application's available surfaces, keep its real engine alive, expose narrowly scoped state/actions, and bind those to movable/composable views. Prefer existing public semantics, then accessibility/DOM, then framework introspection, then authorized runtime instrumentation; retain pixel portals as a rendering/fallback option. The missing product is the common component contract, discovery/verification machinery, lifecycle orchestration and compelling composition UX. The missing primitive is not simply “access to the source.”
