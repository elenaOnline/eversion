> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Composable desktop: research lineage and closest precedents

Research checked 7 October 2026. This memo concerns preserving and composing **existing, distinctive software**, including software whose source is unavailable. It separates working research artifacts, public source, deployed technologies, and architectural proposals. No software was installed or run; implementation claims are based on primary project documentation, papers, and inspected source. A repository is evidence of an implementation, not independent verification that every advertised feature works.

## Bottom line

The desired idea has a substantial, surprisingly direct lineage. **UI Façades, WinCuts, Prefab, Fusion, and Arcan/Durden/Pipeworld** are closer technical precedents than most current “AI desktop” products. **SkyCraft and Minecraft–GTA passthrough** provide the most illuminating recent analogy: keep both real runtimes, selectively share state and outputs, decide who controls which subsystem, and make their interactions coherent. **Webstrates/Varv, Engraft, Cambria, and in-the-wild substrate research** explain what a good composition layer should provide, while also exposing why universally extracting arbitrary app internals is a different and harder claim.

The credible opportunity is not inventing UI remixing. It is combining previously separated layers: runtime-preserving visual composition, discoverable actions and data, explicit identity/ownership relationships, persistent user-authored behavior, intelligent adapter authoring and repair, and honest degradation across heterogeneous applications. None of the sources below establishes a universal, production-ready system delivering all of those properties across arbitrary native and web software.

## 1. Closest literal ancestors: compose real running interfaces

### WinCuts — CHI 2004

**What was built:** Microsoft Research demonstrated independent, interactive windows showing arbitrary regions of existing windows. Regions remained live and could also be shared across devices. This is a direct predecessor of “take just this part of that application and keep using it here.”

**Depth:** pixels plus routed interaction; no claim of a universal document/entity model. **Existing apps:** yes, source-window regions. **Status:** implemented research prototype and published implementation discussion; a maintained downloadable product was not verified. Beware the unrelated modern GitHub project named WinCuts, which manages keyboard shortcuts.

**Lesson:** live clipping is a proven interaction technique, not speculative AI functionality. It solves spatial attention and access, not automatically the relationship between the things those regions represent.

Primary source: [Microsoft Research, WinCuts](https://www.microsoft.com/en-us/research/publication/wincuts-manipulating-arbitrary-window-regions-for-more-effective-use-of-screen-space/)

### User Interface Façades / Metisse — UIST 2006

**What was built:** direct manipulation for adapting, recombining, and duplicating existing GUI elements. The project page identifies a freely available implementation in the Linux-based Metisse window system. The paper describes an offscreen-rendering X server, a separate compositor, redirected input, and accessibility APIs for locating widgets and implementing replacements. A source region can participate in multiple façades. The work explicitly includes interaction adaptation, not only appearances.

**Depth:** live visual/input composition, with richer widget-level behavior where accessibility exposes it. **Existing apps:** yes, without modifying their source. **Platform:** Metisse/X/Linux ecosystem, not a shipping cross-platform shell for today's entire app population. **Status:** real historical implementation with published source-distribution route; present-day buildability was not tested.

**Lesson:** this is likely the closest historical statement of the user's physical interaction concept. The remaining leap is robust modern hosting, semantic relationships, lifecycle and permissions, and adaptation that survives applications changing.

Primary sources: [project](https://ws.iat.sfu.ca/facades/), [UIST paper](https://direction.bordeaux.inria.fr/~roussel/publications/2006-UIST-uifacades.pdf)

### Prefab — CHI 2010 onward; toolkit 2014

**What was built:** a pixel-based reverse-engineering toolkit that recognizes widgets from examples and constructs an interface hierarchy resembling a DOM. It can interpret screenshots repeatedly to support real-time interface modifications. Its implementation is C# on Windows, while remote-desktop pixels allow it to inspect interfaces from other systems. Source remains public.

**Depth:** inferred structure and annotated semantics, with recognition templates rather than guaranteed access to original application state. **Existing apps:** yes; no developer cooperation required for pixel access. **Status:** public research toolkit, not evidence of universal contemporary compatibility.

The later Layers and Annotations work emphasizes reusable interpretations and correction of inferred metadata. That is highly relevant to agent-authored adapters: detection, annotation, and correction should be reusable assets rather than disposable one-shot scripts.

**Lesson:** screenshot-only does not imply “nothing beyond clicking.” Useful structural models can be inferred externally. But a recognized rectangle is not the original domain object, and visible containment differs from logical hierarchy.

Primary sources: [project](https://prefab.github.io/), [code](https://github.com/prefab/code), [2014 paper](https://homes.cs.washington.edu/~jfogarty/publications/uist2014.pdf)

### Fusion — UIST 2018

A particularly close web counterpart: live web interfaces are repurposed as reusable components for opportunistic prototypes and mashups. It addresses lifting existing functionality rather than reimplementing whole products. Its fidelity choices and security compromises are essential to interpreting the demonstrations; the accompanying web-feasibility research covers these in detail.

**Status:** implemented research system and paper. **Boundary:** a convincing prototype under modified browser/proxy conditions is not proof that arbitrary authenticated production websites can be embedded securely without adaptation.

Primary source: [Fusion paper](https://pg.ucsd.edu/publications/Fusion-opportunistic-web-prototyping-UI-mashups_UIST-2018.pdf)

### Arcan, Durden, Pipeworld — source-available experimental desktop lineage

Durden documents **window slicing**, overlays that follow across contexts, and input multicast. Slicing can retain interaction with a selected portion of a window. Arcan provides explicit bridges to Wayland/X11 clients and separates those compatibility processes from its own core.

Pipeworld combines a zooming/tiling desktop with spreadsheet-like, typed dataflow cells. Its public repository describes producer/consumer cells, composition, terminal and application hosting, and operation either as a desktop or inside another desktop. The repository is BSD-3-Clause and candidly warns that it tracks recent Arcan development.

**Depth:** much stronger compositor/dataflow unification than a simple window grid; external app domain semantics still require exposed channels or adapters. **Existing apps:** native Arcan clients plus protocol bridges; not every proprietary application on every OS. **Status:** inspectable source and documented demos, experimental rather than polished universal replacement.

**Lesson:** this is a serious alternative OS/compositor lineage to study, especially if engineering effort is unconstrained. A composition engine can treat windows, streams, shell processes, and data as related computational objects without pretending all are equivalent.

Primary sources: [Durden slicing](https://arcan-fe.com/2017/09/22/arcan-0-5-3-durden-0-3/), [Pipeworld introduction](https://arcan-fe.com/2021/04/12/introducing-pipeworld/), [Pipeworld source](https://github.com/letoram/pipeworld), [Arcan client bridges](https://arcan-fe.com/2020/11/24/arcan-0-6-m-start-networking/)

## 2. Passthrough mods: the analogy is technically substantive

### SkyCraft: real Skyrim plus real Minecraft

The public SkyCraft repository describes an early experimental SKSE plugin plus Fabric mod. Both games retain their respective runtimes: Minecraft supplies its gameplay systems; Skyrim retains its world and NPC systems. The README documents shared memory, hidden Minecraft operation, version-specific prerequisites, conflicts, and limitations. This is genuine runtime reuse as reported by the project, not merely a recreated Minecraft-looking interface. The README also acknowledges missing capabilities and mismatched world behavior; those limitations matter more than the spectacle.

Primary source: [SkyCraft repository/README](https://github.com/chasmlol/SkyCraft)

#### Important distinction: design document is not current code

The design file labels itself draft v0.1, dated 29 September 2026. Its plan assigns authority to each engine, describes collision and actor proxies, translates hit events, routes input based on which menu owns it, and proposes synchronizing rendering and saves. It proposes GPU texture/depth compositing, including fallback paths. Several sections remain explicitly future work or open decisions.

Primary source: [SkyCraft design](https://github.com/chasmlol/SkyCraft/blob/main/docs/DESIGN.md)

The inspected **current protocol header** is more specific and in places materially different. It defines shared state and heartbeats, input events, water grids, actor records, hit events, collision triangles and occupancy data. Crucially, its render protocol transports Minecraft-generated meshes and texture atlases for Skyrim to draw, including avatar/entity geometry and block lighting metadata. Therefore, describe the current system as **state/event/geometry bridging plus rendering reuse**, and do not uncritically repeat the draft's GPU texture-sharing plan as deployed implementation. Header inspection verifies a concrete schema, not runtime correctness of all producers and consumers.

Primary source: [SkyCraft protocol source](https://github.com/chasmlol/SkyCraft/blob/main/protocol/skycraft_protocol.h)

#### What transfers to the desktop idea — analysis

The useful analogy is **co-simulation with a negotiated boundary**:

- Preserve the original engines and specialist behavior
- Export only the information another engine needs
- Use proxies where one engine needs to “feel” an object owned by another
- Define authority per property or subsystem
- Route input according to context, focus, and modal state
- Translate events rather than silently maintaining two unrelated copies
- Decide explicitly how partial failure and persistence work

A desktop equivalent might retain a real CAD viewport, real spreadsheet calculation engine, and real issue-tracker record. A selection in one becomes a typed event or identified reference in the others. The shell can replace surrounding orchestration and presentation without reproducing CAD or spreadsheet semantics.

This also identifies the limit: game modding succeeds because the bridge gains precise hooks into particular runtimes and establishes particular mappings. It does not prove a universal translator for arbitrary opaque programs. More effort can find more hooks; it cannot remove the need for hooks, consent, identity, or meaningful mappings.

### Minecraft × GTA V passthrough

The example repository describes two real games running concurrently: camera/ground/input coordination over local WebSocket, exported Minecraft color/depth and overlay data through shared memory, and a ReShade compositor testing against GTA depth. Events also cross the boundary: block placement produces GTA collision props; explosions and weapons produce corresponding GTA effects. The code was documented as tested against a specific GTA Legacy build in September 2026.

This is a complementary implementation strategy to SkyCraft's current mesh-oriented protocol. Neither is a generic source-code merger. **“Passthrough mod” is best treated here as this concrete family's descriptive term, not an established universal technical standard.** These are strong likely referents for the user's phrase, but only the user can confirm their intended example.

Primary source: [Minecraft–GTA passthrough example](https://github.com/rehan-remade/universal-modder/blob/main/examples/minecraft-gta5-passthrough/README.md)

## 3. Web augmentation and end-user integration before agents

### Chickenfoot — MIT, UIST 2005 and later

Chickenfoot let users automate, customize, and integrate web applications from inside Firefox, using rendered-page concepts and keyword descriptions rather than requiring HTML inspection. Examples included navigation, form operations, content extraction, and insertion. Its in-browser positioning deliberately preserved the user's sessions and rendered context.

**Depth:** behavior/data extraction and augmentation through browser primitives. **Existing apps:** yes, web. **Status:** historically implemented research extension; compatibility with current Firefox was not verified.

**Lesson:** user intent can be expressed at the level of “that button/that field” without first standardizing the whole site. Modern models improve inference and authoring, but the basic goal predates them.

Primary sources: [MIT chapter](https://dspace.mit.edu/entities/publication/7660ecb6-fc37-455d-9b5a-ca310d0358a0), [original project account](https://publications.csail.mit.edu/abstracts/abstracts06/rcm/rcm.html)

### d.mix — Stanford, UIST 2007

Users sampled visible elements of annotated sites; the system generated the service calls corresponding to those examples and placed them in an editable wiki-based environment. Knowledgeable developers supplied site-to-service maps. The original site need not cooperate with d.mix itself, but usable underlying services and mappings were still required.

**Depth:** semantic/API reconstruction of sampled content, not capture of an arbitrary original widget runtime. **Status:** research prototype and small user study.

**Lesson:** selecting the source in situ and then offering the best available implementation is a valuable interaction. The hidden adapter labor must not be mistaken for universal automatic interoperability.

Primary sources: [project](https://hci.stanford.edu/research/mashups/), [paper](https://hci.stanford.edu/cstr/reports/2007-09.pdf)

### WebMakeup — visual web augmentation

This research tool lets users move/remove nodes and add material from different web pages. Its thesis explicitly addresses locator fragility after website updates and introduces alternative locators. It is a valuable precedent for persistent user-defined modifications, including the maintenance problem.

**Depth:** DOM-based augmentation; web-only. **Status:** implemented Chrome-extension research tool documented in thesis; current distribution was not verified.

Primary source: [author's university thesis record](https://ekoizpen-zientifikoa.ehu.eus/documentos/5ecb7f7c2999521315203e3b)

### Wildcard — spreadsheet-driven browser customization

Wildcard exposes a simplified table view of supported web-app data for user modification, annotations, and spreadsheet-style transformations. Its public MIT-licensed source is explicitly pre-release; the README warns that some advertised features, including filtering and formulas, were not yet ported to its current branch.

**Depth:** extracted/adapter-mediated structured data with page customization. **Existing apps:** yes, supported websites; not any site without adaptation. **Status:** genuine public research code with explicit incompleteness.

**Lesson:** distinguish a paper's demonstrated feature set from the repository branch available today. A sheet is a plausible relationship editor for nonprogrammers, but making everything tabular can erase source-specific strengths.

Primary sources: [project paper](https://www.geoffreylitt.com/wildcard/salon2020/), [repository](https://github.com/geoffreylitt/wildcard)

### TabFS — live browser state as ordinary files

TabFS connects a browser extension to a FUSE filesystem, making browser state accessible through ordinary filesystem tools. The native component forwards requests to the extension. It has public GPL-3.0 source.

**Depth:** browser-state/control interoperability, not a visual fragment host. **Existing apps:** browser tabs, within extension capabilities. **Lesson:** make a useful slice of an existing runtime speak an already composable protocol instead of replacing that runtime.

Primary sources: [author's explanation](https://omar.website/tabfs/), [source](https://github.com/osnr/TabFS)

## 4. Native component and hyperdocument ancestry

### OLE / COM compound documents

OLE is deployed technology for placing objects made by different applications into one document and activating the original editing facilities in context. Microsoft's documentation makes the required COM, persistence, storage, and data-transfer interfaces explicit.

**Depth:** real embedded/linked objects and original editors, often considerably deeper than pixels. **Existing apps:** participating applications, not arbitrary unmodified software. **Status:** documented platform technology, not just a concept.

**Lesson:** preserving specialist engines inside a composed experience has shipped for decades. The historical catch is the participation contract and lifecycle complexity, not conceptual impossibility.

Primary source: [Microsoft compound documents](https://learn.microsoft.com/en-us/windows/win32/com/compound-documents)

### OpenDoc

Apple's historical developer material describes part editors and viewers whose functionality appears inside compound documents. This is the stronger document-centric component vision: obtain the editing capability appropriate to a part instead of treating the whole application as the indivisible unit.

**Depth:** participant components; **status:** historical implemented platform, not a current adoption recommendation. It did not automatically decompose all legacy applications.

Primary source: [Apple develop article, preserved](https://preserve.mactech.com/articles/develop/issue_22/opendoc.html)

### KDE KParts

KParts reuses GUI components, with viewer/editor parts and optional browser/text/scripting interfaces. This is another concrete, inspectable native component ecosystem.

**Depth:** strong within supported component contracts. **Constraint:** software must expose a KPart or compatible component; it is not arbitrary external window surgery.

Primary sources: [KDE tutorial](https://techbase.kde.org/Development/Tutorials/Using_KParts), [interfaces](https://api.kde.org/legacy/4.14-api/kdelibs-apidocs/kparts/html/annotated.html)

### Engelbart's Open Hyperdocument System

OHS frames fine-grained addressability, transclusion, alternate views, provenance, and cross-vendor collaboration as foundational infrastructure. It is not merely an infinite canvas. The institute distinguishes the broad framework and evolving prototypes from a fully realized universal deployment.

**Lesson:** stable links and relationships between live work objects are at least as important as moving visual regions. A shell that lacks object identity repeats the weaker aspects of windows even if it looks novel.

Primary source: [OHS overview](https://dougengelbart.org/content/view/156/)

### Plan 9 plumbing

The plumber routes context-sensitive messages to appropriate tools, such as taking a compiler error to the matching file/line. It demonstrates cross-tool relationships that are neither a monolithic app nor a dashboard.

**Lesson:** composition can be a small, inspectable routing rule. A modern relationship layer need not always generate a new interface.

Primary source: [Plan 9 plumbing examples](https://9p.io/sources/wiki/d/18.hist)

## 5. Malleable substrates: deep composition, with an adoption boundary

### Webstrates / Codestrates / Varv

Webstrates persists and synchronizes DOM changes, including code, and supports transclusion. Codestrates supplies authoring/execution tools. Varv expresses interactive software through declarative concepts, schemas, and actions; extensions can add or override behavior incrementally. Its demonstrations include recombining game rules and adapting views.

These projects provide real public source, and Webstrates organization activity includes updates into 2026. Varv's documentation distinguishes its normal Webstrates workflow from an early Electron proof of concept that is not offered as a release.

**Depth:** high inside the model, including state and behavior. **Boundary:** the source must participate in this environment or be adapted; a hosted third-party SaaS does not become a webstrate merely because it uses a DOM.

**Lesson:** excellent model for the shell's own relationship/extension layer, but not evidence that all existing software is already decomposable.

Primary sources: [Webstrates publications](https://webstrates.net/project/publications/), [Varv paper](https://vis.mit.edu/pubs/varv/), [Varv usage/status](https://varv.projects.cavi.au.dk/docs/usage/), [public projects](https://github.com/Webstrates)

### Potluck and Embark

Potluck gradually enriches text documents with computations and interactive behavior. Embark uses more structured outlines, typed mentions such as places/dates, rich maps/calendars, and computations for travel planning. Both are explicitly research prototypes.

**Depth:** meaningful relationships and computations in a new authoring substrate. **Existing apps:** integrations/data sources, not arbitrary native app surfaces. **Lesson:** good evidence for contextual, living relationships and progressive enrichment. Poor evidence for “retain all my existing unique interfaces unchanged.”

Primary sources: [Potluck](https://www.inkandswitch.com/potluck/), [Embark](https://www.inkandswitch.com/embark/)

### Engraft — UIST 2023

Engraft offers an API for recursively embedding live, rich programming tools in other tools and hosts. It is public source, with a paper and demonstrations; documentation is described as limited.

**Depth:** composable programmable components, not magical extraction from opaque apps. **Lesson:** a good precedent for connecting agent-authored glue and user-editable relationships to existing environments without requiring every computation to be plain text code.

Primary sources: [project](https://engraft.dev/), [paper](https://engraft.dev/engraft-uist-2023.pdf)

### Dash — Brown hypermedia system

Brown's Dash is a browser-based collaborative environment for spatial collections, fine-grained links, metadata, multimedia documents, and multiple composable views. Its documentation describes an online service and classroom use.

**Depth:** common document/relationship model. **Boundary:** reuses and organizes heterogeneous media through Dash's implementation; it is not evidence of arbitrary third-party application behavior being transplanted. Do not confuse it with Plotly Dash.

Primary sources: [Dash documentation](https://brown-dash.github.io/Dash-Documentation/about/), [architecture by its developers](https://hackmd.io/o7giQJUmTQ6mtyL3id0EVA)

### Polyphony — Raffaillac and Huot, EICS 2019

The relevant Polyphony is an experimental GUI toolkit using Entity–Component–System design: entities carry data components, and reusable systems select and act on them. This loosens the traditional widget/object hierarchy.

**Depth:** internal toolkit architecture. **Boundary:** new or adapted software must be built in that model; it does not extract arbitrary GUI components. Paper and author thesis verified; a current supported release was not verified. There are numerous unrelated tools named Polyphony.

Primary source: [author's thesis](https://traffaillac.github.io/content/manuscrit.pdf); paper DOI: https://doi.org/10.1145/3331150

## 6. The less visible issue: preserving meaning across tool boundaries

### Cambria — bidirectional edit lenses

Cambria implements schema transformations as composable bidirectional lenses, with an experimental collaborative issue tracker. Its authors explicitly demonstrate incompatibility tradeoffs: a single-assignee model and a multi-assignee model cannot always preserve every desirable property simultaneously. Their design retains schema-tagged original writes and translates for readers rather than forcing one universal schema.

**Status:** implemented TypeScript research library and prototype. **Lesson:** a relationship graph should retain source-native representations and make losses explicit. “AI understands both” does not make a lossy mapping invertible. This is a fundamental semantic constraint, not merely an engineering cost.

Primary source: [Cambria](https://www.inkandswitch.com/cambria/)

### In-the-wild substrates — Trividic, 2025

This position paper explicitly argues for making the existing heterogeneous software ecology more interoperable instead of imposing an unfamiliar replacement workflow. It grounds this in publishing: Propage supports multiformat outputs and linked views; OutDesign helps route InDesign's IDML through Pandoc. The work includes contributions to WeasyPrint and Pandoc. The paper distinguishes practical conversion from the much harder goal of full bidirectional synchronization.

**Lesson:** exceptionally close philosophical match to preserving unique tools and expert skill. This is a principled research approach, not a compromise to abandon once agents can generate code.

Primary source: [Synchronising Content Across Formats In-the-wild](https://software-substrates.github.io/proceedings/2025/statements/Paper%2010%20-%20Yann%20Trividic%20-%20Synchronising%20Content%20Across%20Formats%20In-the-wild.pdf)

## 7. Product and research conclusions — synthesis, not source claims

### A. A single fidelity scale is misleading

At least four independent axes matter:

1. **Presentation fidelity:** is this the real original view, a faithful rendering of exported objects, or a new approximation?
2. **Behavior fidelity:** are real original handlers/engines running, or is the shell emulating selected actions?
3. **Semantic fidelity:** does the fragment preserve domain identity, state, relationships, and invariants?
4. **Continuity fidelity:** does it survive reloads, app updates, offscreen states, authentication changes, and concurrent edits?

A cropped original CAD viewport may have excellent visual/behavior fidelity and little exposed semantics. A clean API-backed task card may have excellent identity and updates but none of the original rich editor's behavior. Neither should be mislabeled “the real app” without qualification.

### B. The best architecture can be heterogeneous on purpose

For each fragment, negotiate the strongest available boundary: official embedded component; plugin/API-backed real entity; retained original DOM/runtime; accessibility-linked native view; interactive surface capture; read-only screenshot as a visibly degraded fallback. The relationship object above these should say what it knows, what it can do, where authority resides, and how fresh its state is. This avoids the false choice between universal source access and rebuilding everything.

### C. The novel product center is the relationship, not the tile

The most useful persisted unit might be: “this source selection identifies these entities; display their live source controls here; this event updates this field there; before consequential writes show this preview; if the source changes revalidate this mapping.” A layout is one projection of those relationships. This leaves room for opinionated, autonomous behavior without turning the concept into a generic AI dashboard.

### D. Intelligent repair is an opportunity, not a correctness proof

Agents could author selectors, infer DOM/accessibility patterns, generate transformations, locate API equivalents, notice divergence, and propose or test repairs. Durable compositions should retain tests, examples, provenance, version constraints, ownership, rollback, and permission boundaries. Silent repair can be worse than breakage if a button or field acquires a different meaning.

### E. Preserve hard limits even with unlimited engineering

- A visible interface does not expose every hidden state or invariant
- Some data mappings are lossy or underspecified
- Two independent authorities can disagree; distributed updates need conflict and failure behavior
- The original runtime may stop, log out, change shape, or forbid capture/embedding
- Security isolation and authentication are boundaries to preserve, not annoyances to disable globally
- Input focus, modal dialogs, IME, accessibility, drag/drop, and undo belong to the composed interaction contract
- Source-specific permissions remain attached to operations, even when their controls look native to the shell

### F. A credible evaluation should use adversarially varied real tools

Evaluate more than a curated set of cooperating demos: a complex authenticated SaaS, custom canvas/WebGL tool, conventional accessible native app, poorly accessible native app, rich document editor, terminal, and source-available extensible tool. Record which fidelity properties survive, not just whether a screenshot looks seamless. Include reload, resize, update, sign-out, network loss, concurrent edits, and partial writes.

## 8. Names, uncertainty, and false friends

- **Metamuse** resolves in this research lineage to Muse's tools-for-thought podcast, not a verified composition platform. There are unrelated marketing, algorithm-generation, and other products using similar names. Do not list “Metamuse” as a technical predecessor without identifying the intended referent. [Muse's own episode page](https://allume.com/podcast/1-tool-switching/)
- **Polyphony** here means Raffaillac/Huot's ECS GUI toolkit, not a synthesis compiler or music application
- **Dash** here means Brown's hypermedia environment, not Plotly
- **Prefab** here means Dixon/Fogarty's pixel reverse-engineering toolkit, not current generative-UI libraries bearing that name
- **WinCuts** here means the Microsoft Research window-region system, not the unrelated shortcut utility
- “Pass-through mods” now has concrete likely matches, but the phrase alone does not specify which rendering/state strategy is used

## Suggested reading order

1. UI Façades paper: closest historical GUI interaction
2. SkyCraft protocol + README, then its draft design: strongest current runtime-reuse analogy and a caution against documentation drift
3. Fusion: opportunistic web extraction and the fidelity/security boundary
4. Arcan/Durden/Pipeworld: compositor plus dataflow architecture
5. Cambria: semantic limits and source-native state
6. Trividic 2025: preserve the existing ecology and expertise
7. Webstrates/Varv and Engraft: make the new glue genuinely malleable
8. Potluck/Embark and OHS: design for contextual relationships rather than just containers
