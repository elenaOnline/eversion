# Modding: evidence and documented mechanisms

Editorial extraction, 2026-10-07. Source research was read-only; no listed software was installed, executed, or benchmarked. Documented APIs, author-reported demonstrations, source inspections, and untested compatibility are distinguished below. Source-level conclusions are limited to the investigated interfaces and versions. No implementation architecture is prescribed here.

## 1. The game analogy is real and is richer than image overlay

<!-- Origin: modding.md lines 11–11 -->

### Actual GTA/Minecraft passthrough projects

<!-- Origin: modding.md lines 13–13 -->

The GTA V Enhanced × Minecraft Steve project documents two games running simultaneously: a ScriptHookV plugin supplies GTA camera/collision data, a Fabric mod runs Minecraft, and ReShade depth-composites the Minecraft rendering. Minecraft events become GTA effects: explosions, arrows, movement, and block collision. It is explicitly tied to specific game versions and warns that GTA updates break ASI compatibility. This is not equivalent to a universal importer; it is a purpose-built bilateral adapter. [Project README](https://github.com/utku6767/Gta-V-Enhanced-Minecraft-Steve-Passthrough/blob/main/README.md)

<!-- Origin: modding.md lines 15–15 -->

A more explicit implementation, Wither Storm GTA passthrough, documents local WebSocket exchange of camera, player, input, and ground samples; Minecraft exports color, depth, and HUD/hand layers via a Windows shared-memory mapping; a ReShade add-on uploads and composites them. The Forge bridge reports actions such as entity grabs and terrain consumption; GTA implements corresponding forces, removals and effects. Its limitations include non-destructible native terrain/buildings, GPU/CPU cost, and dependency on a usable depth buffer. [Repository](https://github.com/VortexisTV/wither-storm-gta5-passthrough)

<!-- Origin: modding.md lines 17–17 -->

The upstream Universal Modder repo explicitly distinguishes passthrough mods from asset ports, decompositions and reimplementations, and includes a Minecraft–GTA example. It uses game recon followed by engine-specific tooling, documented field notes, and a verified working slice; its existence does not establish automatic support for every game. [Upstream](https://github.com/rehan-remade/universal-modder)

<!-- Origin: modding.md lines 19–19 -->

Important false-positive: GrandTheftMinecraft is a native GTA mod recreating Minecraft-like behavior with GTA objects and locally sourced Minecraft assets. It is a different architecture from concurrently running Minecraft and synchronizing its engine. [GrandTheftMinecraft](https://github.com/cyteon/GrandTheftMinecraft)

<!-- Origin: modding.md lines 21–21 -->

The exact bare project name “MinecraftGTA” was not resolved to a unique primary repository in this search. Use the explicit verified project names above; do not silently conflate them. SkyCraft is covered by the other research stream.

<!-- Origin: modding.md lines 23–23 -->

## 2. Existing runtime instrumentation can expose functionality

<!-- Origin: modding.md lines 38–38 -->

### Frida

<!-- Origin: modding.md lines 40–40 -->

Frida documents interception/replacement of native functions, calling native functions by address and ABI, runtime-specific bridges, and an RPC export mechanism to a controlling program. Therefore a custom adapter can expose discovered functions to a separate UI and observe relevant calls. Frida does not infer a stable domain API automatically: the integrator still needs correct argument types, object lifetime, thread rules and semantics. It also explicitly reports code-signing policy and whether its interception facilities are available. [JavaScript API](https://frida.re/docs/javascript-api/)

<!-- Origin: modding.md lines 42–42 -->

### DynamoRIO

<!-- Origin: modding.md lines 46–46 -->

DynamoRIO can transform an unmodified program's running instruction stream, not merely insert tracing callbacks. It supports stock Windows, Linux and Android, with experimental macOS support. Its code manipulation API documents transformations and real restrictions, but does not provide a semantic desktop framework. [Project](https://dynamorio.org/), [Code manipulation API](https://dynamorio.org/API_BT.html)

<!-- Origin: modding.md lines 48–48 -->

### Microsoft Detours

<!-- Origin: modding.md lines 52–52 -->

Detours dynamically redirects binary functions to replacement functions, preserving the original implementation through a trampoline. It changes in-memory execution rather than requiring the application's original source. It is a proven interception primitive, not an automatic UI export system. [Microsoft overview](https://github.com/microsoft/detours/wiki)

<!-- Origin: modding.md lines 54–54 -->

### Windhawk

<!-- Origin: modding.md lines 56–56 -->

Windhawk is an existing Windows mod distribution and lifecycle system. A mod is C++ compiled to a DLL and loaded in each targeted process; its APIs install function hooks. Symbol utilities include binary-version-specific caching and typed wrappers. [Mod development](https://github.com/ramensoftware/windhawk/wiki/Creating-a-new-mod), [Development tips](https://github.com/ramensoftware/windhawk/wiki/Development-tips), [Project](https://github.com/ramensoftware/windhawk)

<!-- Origin: modding.md lines 58–58 -->

## 3. Framework introspection tools

<!-- Origin: modding.md lines 62–62 -->

These examples show that one need not choose only between public APIs and arbitrary binary reverse engineering.

<!-- Origin: modding.md lines 64–64 -->

### GammaRay: Qt

<!-- Origin: modding.md lines 66–66 -->

GammaRay can browse live QObject trees, edit properties, invoke slots, monitor signals, and inspect signal/slot connections. It also understands higher-level Qt models, state machines and scene graphs. [Primary repository](https://github.com/KDAB/GammaRay)

<!-- Origin: modding.md lines 68–68 -->

It can attach to an already-running process or launch with a probe. Its launcher identifies Qt versions and matching probes, and refuses unavailable ABI combinations. This supplies a concrete model for runtime stack detection plus compatible adapter selection. [Launcher documentation](https://docs.kdab.com/gammaray-manual/latest/gammaray-launcher-gui.html), [CLI](https://docs.kdab.com/gammaray-manual/latest/gammaray-command-line.html)

<!-- Origin: modding.md lines 70–70 -->

### Snoop: WPF

<!-- Origin: modding.md lines 72–72 -->

Snoop inspects visual, logical and automation trees in running WPF applications without requiring a debugger; it changes properties, inspects triggers and watches property changes. [Project](https://github.com/snoopwpf/snoopwpf)

<!-- Origin: modding.md lines 74–74 -->

### UWPSpy: UWP/WinUI 3

<!-- Origin: modding.md lines 76–76 -->

UWPSpy inspects and manipulates running UI elements and properties. It is a framework-specific source-independent view into the actual widget tree. [Project](https://github.com/m417z/UWPSpy)

<!-- Origin: modding.md lines 78–78 -->

## 4. Semantic access without injection

<!-- Origin: modding.md lines 82–82 -->

### Accessibility is already a partial capability protocol

<!-- Origin: modding.md lines 84–84 -->

Windows UI Automation control patterns expose methods, properties, relationships and events for meaningful behaviors, including Invoke, Value, Selection, Grid and Scroll. One element can implement several patterns. This can drive a substitute frontend without replicating all original screen interactions. [Microsoft control patterns](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-controlpatternsoverview)

<!-- Origin: modding.md lines 86–86 -->

On macOS, Hammerspoon's AX wrapper reads and writes attributes and invokes supported actions; observers subscribe to UI changes. Its documentation is candid that applications decide what they expose and may decline changes or report success without the intended effect. [AX module](https://www.hammerspoon.org/docs/hs.axuielement.html), [Observers](https://www.hammerspoon.org/docs/hs.axuielement.observer.html)

<!-- Origin: modding.md lines 88–88 -->

### Browser extensions/userscripts

<!-- Origin: modding.md lines 92–92 -->

Chrome content scripts can inspect/modify a page DOM and communicate with an extension. Their default JavaScript world is isolated from the site's JavaScript; the MAIN execution world shares the page's environment and consequently its interference risks. These are real mechanisms for modifying a closed-source website's live client UI. [Content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts), [Execution-world manifest setting](https://developer.chrome.com/docs/extensions/reference/manifest/content-scripts)

<!-- Origin: modding.md lines 94–94 -->

### Webfuse: live web virtualization/proxy

<!-- Origin: modding.md lines 98–98 -->

Webfuse documents an augmented proxy, rewritten requests/DOM, JS sandbox and session extension environment that changes a live site without needing its source. Its automation API produces DOM snapshots, including cross-frame/shadow traversal and stable Webfuse element IDs, and supports actions against those targets. This is an existing vendor architecture adjacent to source-independent web composition; broad compatibility and performance claims are not independently established here. [Architecture](https://www.webfuse.com/blog/web-augmentation), [Automation API](https://dev.webfuse.com/automation-api/)

<!-- Origin: modding.md lines 100–100 -->

## 5. Automation, history, and interface tools

<!-- Origin: modding.md lines 104–104 -->

- **Terminator / Mediar:** existing Windows-only automation SDK/MCP with element locators, workflow recording and browser extension support. It combines pixels, DOM and accessibility. The repository claims background interaction, but app-specific behavior must be verified; headline speed/success claims are vendor claims. It provides an actuation layer, not a spatial composer. [Repository](https://github.com/mediar-ai/terminator)
- **Microsoft UFO:** a documented hybrid GUI/API agent architecture mixes UIA with native COM/REST/MCP capabilities and chooses per step. It is workflow automation rather than extracted live component composition. [Hybrid action layer](https://github.com/microsoft/UFO/blob/main/documents/docs/ufo2/core_features/hybrid_actions.md)
- **Screenpipe:** current repository emphasizes local screen/audio history, workflow mapping and agent context. It is source-available, with separate privacy implications for cloud integrations. It can supply observation/context; its current README does not establish live functionality extraction or arbitrary UI transplantation. [Repository](https://github.com/screenpipe/screenpipe)
- **Rewind:** historical capture/search precedent only. Limitless says the latest Rewind update disables screen/audio capture starting December 19, 2025. [Official announcement/FAQ](https://www.limitless.ai/)
- **OnTopReplica:** genuine live window/subregion cloning with DWM thumbnails and click forwarding. This demonstrates useful cropped portals, but issue reports show app-specific click behavior and focus-stealing. It does not expose document semantics. [README](https://github.com/LorenzCK/OnTopReplica/blob/master/README.md), [Focus issue](https://github.com/LorenzCK/OnTopReplica/issues/65)
- **Prefab (Dixon/Fogarty):** direct research precedent for runtime interface modification without source or application cooperation using pixel-level reverse engineering. Its published work includes recovering interface content/hierarchy and advanced interaction behaviors. It establishes the lineage of semantic reconstruction from visuals, not a currently universal production composer. [Project/papers](https://prefab.github.io/papers.html), [University talk](https://engineering.oregonstate.edu/colloquium/1327)

<!-- Origin: modding.md lines 106–111 -->

### Genuine boundaries

<!-- Origin: modding.md lines 121–121 -->

- **Remote hidden state:** a local hook observes what reaches the client. It cannot reveal server-only records, private computations or unauthorized capabilities absent from the session. This is an architectural inference, not a claim about a particular service.
- **OS security:** Windows protected processes restrict memory access and thread creation. Apple's hardened runtime normally constrains loaded libraries by signatures/team identity. [Windows process rights](https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights), [Apple library validation](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.cs.disable-library-validation)
- **ABI and lifecycle:** a function address and signature are not portable API guarantees. Compiler/runtime updates, object destruction, UI thread affinity and reentrant calls matter. GammaRay's probe matching is a concrete example of engineering this boundary rather than pretending it disappears.
- **Licensing/distribution:** inspect each chosen dependency and application's terms before packaging. Permission to run software, permission to modify one's running instance, and permission to redistribute code/assets are separate questions. This memo makes no jurisdiction-specific legal conclusion.

<!-- Origin: modding.md lines 123–127 -->
