# Precedents: evidence and documented mechanisms

Editorial extraction, 2026-10-07. Source research was read-only; no listed software was installed, executed, or benchmarked. Documented APIs, author-reported demonstrations, source inspections, and untested compatibility are distinguished below. Source-level conclusions are limited to the investigated interfaces and versions. No implementation architecture is prescribed here.

### WinCuts — CHI 2004

<!-- Origin: precedents.md lines 13–13 -->

**What was built:** Microsoft Research demonstrated independent, interactive windows showing arbitrary regions of existing windows. Local regions remained live and interactive; the remote-sharing extension was read-only. The latter qualification corrects the more general wording of the original precedent memo; see report.md and its WinCuts paper link.

<!-- Origin: precedents.md lines 15–15 -->

**Depth:** pixels plus routed interaction; no claim of a universal document/entity model. **Existing apps:** yes, source-window regions. **Status:** implemented research prototype and published implementation discussion; a maintained downloadable product was not verified. Beware the unrelated modern GitHub project named WinCuts, which manages keyboard shortcuts.

<!-- Origin: precedents.md lines 17–17 -->

Primary source: [Microsoft Research, WinCuts](https://www.microsoft.com/en-us/research/publication/wincuts-manipulating-arbitrary-window-regions-for-more-effective-use-of-screen-space/)

<!-- Origin: precedents.md lines 21–21 -->

### User Interface Façades / Metisse — UIST 2006

<!-- Origin: precedents.md lines 23–23 -->

**What was built:** direct manipulation for adapting, recombining, and duplicating existing GUI elements. The project page identifies a freely available implementation in the Linux-based Metisse window system. The paper describes an offscreen-rendering X server, a separate compositor, redirected input, and accessibility APIs for locating widgets and implementing replacements. A source region can participate in multiple façades. The work explicitly includes interaction adaptation, not only appearances.

<!-- Origin: precedents.md lines 25–25 -->

**Depth:** live visual/input composition, with richer widget-level behavior where accessibility exposes it. **Existing apps:** yes, without modifying their source. **Platform:** Metisse/X/Linux ecosystem, not a shipping cross-platform shell for today's entire app population. **Status:** real historical implementation with published source-distribution route; present-day buildability was not tested.

<!-- Origin: precedents.md lines 27–27 -->

Primary sources: [project](https://ws.iat.sfu.ca/facades/), [UIST paper](https://direction.bordeaux.inria.fr/~roussel/publications/2006-UIST-uifacades.pdf)

<!-- Origin: precedents.md lines 31–31 -->

### Prefab — CHI 2010 onward; toolkit 2014

<!-- Origin: precedents.md lines 33–33 -->

**What was built:** a pixel-based reverse-engineering toolkit that recognizes widgets from examples and constructs an interface hierarchy resembling a DOM. It can interpret screenshots repeatedly to support real-time interface modifications. Its implementation is C# on Windows, while remote-desktop pixels allow it to inspect interfaces from other systems. Source remains public.

<!-- Origin: precedents.md lines 35–35 -->

**Depth:** inferred structure and annotated semantics, with recognition templates rather than guaranteed access to original application state. **Existing apps:** yes; no developer cooperation required for pixel access. **Status:** public research toolkit, not evidence of universal contemporary compatibility.

<!-- Origin: precedents.md lines 37–37 -->

Primary sources: [project](https://prefab.github.io/), [code](https://github.com/prefab/code), [2014 paper](https://homes.cs.washington.edu/~jfogarty/publications/uist2014.pdf)

<!-- Origin: precedents.md lines 43–43 -->

### Fusion — UIST 2018

<!-- Origin: precedents.md lines 45–45 -->

Fusion: live web interfaces are repurposed as reusable components for opportunistic prototypes and mashups. It addresses lifting existing functionality rather than reimplementing whole products. Its demonstration scope and browser/proxy setup are described in the web findings.

<!-- Origin: precedents.md lines 47–47 -->

**Status:** implemented research system and paper. **Boundary:** a convincing prototype under modified browser/proxy conditions is not proof that arbitrary authenticated production websites can be embedded securely without adaptation.

<!-- Origin: precedents.md lines 49–49 -->

Primary source: [Fusion paper](https://pg.ucsd.edu/publications/Fusion-opportunistic-web-prototyping-UI-mashups_UIST-2018.pdf)

<!-- Origin: precedents.md lines 51–51 -->

### Arcan, Durden, Pipeworld — source-available experimental desktop lineage

<!-- Origin: precedents.md lines 53–53 -->

Durden documents **window slicing**, overlays that follow across contexts, and input multicast. Slicing can retain interaction with a selected portion of a window. Arcan provides explicit bridges to Wayland/X11 clients and separates those compatibility processes from its own core.

<!-- Origin: precedents.md lines 55–55 -->

Pipeworld combines a zooming/tiling desktop with spreadsheet-like, typed dataflow cells. Its public repository describes producer/consumer cells, composition, terminal and application hosting, and operation either as a desktop or inside another desktop. The repository is BSD-3-Clause and candidly warns that it tracks recent Arcan development.

<!-- Origin: precedents.md lines 57–57 -->

**Depth:** compositor/dataflow integration; external app domain semantics still require exposed channels or adapters. **Existing apps:** native Arcan clients plus protocol bridges; not every proprietary application on every OS. **Status:** inspectable source and documented demos, experimental rather than polished universal replacement.

<!-- Origin: precedents.md lines 59–59 -->

Primary sources: [Durden slicing](https://arcan-fe.com/2017/09/22/arcan-0-5-3-durden-0-3/), [Pipeworld introduction](https://arcan-fe.com/2021/04/12/introducing-pipeworld/), [Pipeworld source](https://github.com/letoram/pipeworld), [Arcan client bridges](https://arcan-fe.com/2020/11/24/arcan-0-6-m-start-networking/)

<!-- Origin: precedents.md lines 63–63 -->

## 2. Passthrough mod implementations

<!-- Origin: precedents.md lines 65–65 -->

### SkyCraft: real Skyrim plus real Minecraft

<!-- Origin: precedents.md lines 67–67 -->

The public SkyCraft repository describes an early experimental SKSE plugin plus Fabric mod. Both games retain their respective runtimes: Minecraft supplies its gameplay systems; Skyrim retains its world and NPC systems. The README documents shared memory, hidden Minecraft operation, version-specific prerequisites, conflicts, and limitations. This is genuine runtime reuse as reported by the project, not merely a recreated Minecraft-looking interface. The README also acknowledges missing capabilities and mismatched world behavior; these limit reported functionality.

<!-- Origin: precedents.md lines 69–69 -->

Primary source: [SkyCraft repository/README](https://github.com/chasmlol/SkyCraft)

<!-- Origin: precedents.md lines 71–71 -->

#### Important distinction: design document is not current code

<!-- Origin: precedents.md lines 73–73 -->

The design file labels itself draft v0.1, dated 29 September 2026. Its plan assigns authority to each engine, describes collision and actor proxies, translates hit events, routes input based on which menu owns it, and proposes synchronizing rendering and saves. It proposes GPU texture/depth compositing, including fallback paths. Several sections remain explicitly future work or open decisions.

<!-- Origin: precedents.md lines 75–75 -->

Primary source: [SkyCraft design](https://github.com/chasmlol/SkyCraft/blob/main/docs/DESIGN.md)

<!-- Origin: precedents.md lines 77–77 -->

The inspected **current protocol header** is more specific and in places materially different. It defines shared state and heartbeats, input events, water grids, actor records, hit events, collision triangles and occupancy data. Crucially, its render protocol transports Minecraft-generated meshes and texture atlases for Skyrim to draw, including avatar/entity geometry and block lighting metadata. The header documents state/event/geometry bridging and rendering reuse; its inspected schema differs from the draft GPU texture-sharing plan. Header inspection verifies a concrete schema, not runtime correctness of all producers and consumers.

<!-- Origin: precedents.md lines 79–79 -->

Primary source: [SkyCraft protocol source](https://github.com/chasmlol/SkyCraft/blob/main/protocol/skycraft_protocol.h)

<!-- Origin: precedents.md lines 81–81 -->

### Minecraft × GTA V passthrough

<!-- Origin: precedents.md lines 99–99 -->

The example repository describes two real games running concurrently: camera/ground/input coordination over local WebSocket, exported Minecraft color/depth and overlay data through shared memory, and a ReShade compositor testing against GTA depth. Events also cross the boundary: block placement produces GTA collision props; explosions and weapons produce corresponding GTA effects. The code was documented as tested against a specific GTA Legacy build in September 2026.

<!-- Origin: precedents.md lines 101–101 -->

This is a complementary implementation strategy to SkyCraft's current mesh-oriented protocol. Neither is a generic source-code merger. **“Passthrough mod” is best treated here as this concrete family's descriptive term, not an established universal technical standard.**

<!-- Origin: precedents.md lines 103–103 -->

Primary source: [Minecraft–GTA passthrough example](https://github.com/rehan-remade/universal-modder/blob/main/examples/minecraft-gta5-passthrough/README.md)

<!-- Origin: precedents.md lines 105–105 -->

## 3. Web augmentation and end-user integration before agents

<!-- Origin: precedents.md lines 107–107 -->

### Chickenfoot — MIT, UIST 2005 and later

<!-- Origin: precedents.md lines 109–109 -->

Chickenfoot let users automate, customize, and integrate web applications from inside Firefox, using rendered-page concepts and keyword descriptions rather than requiring HTML inspection. Examples included navigation, form operations, content extraction, and insertion. Its in-browser positioning deliberately preserved the user's sessions and rendered context.

<!-- Origin: precedents.md lines 111–111 -->

**Depth:** behavior/data extraction and augmentation through browser primitives. **Existing apps:** yes, web. **Status:** historically implemented research extension; compatibility with current Firefox was not verified.

<!-- Origin: precedents.md lines 113–113 -->

Primary sources: [MIT chapter](https://dspace.mit.edu/entities/publication/7660ecb6-fc37-455d-9b5a-ca310d0358a0), [original project account](https://publications.csail.mit.edu/abstracts/abstracts06/rcm/rcm.html)

<!-- Origin: precedents.md lines 117–117 -->

### d.mix — Stanford, UIST 2007

<!-- Origin: precedents.md lines 119–119 -->

Users sampled visible elements of annotated sites; the system generated the service calls corresponding to those examples and placed them in an editable wiki-based environment. Knowledgeable developers supplied site-to-service maps. The original site need not cooperate with d.mix itself, but usable underlying services and mappings were still required.

<!-- Origin: precedents.md lines 121–121 -->

**Depth:** semantic/API reconstruction of sampled content, not capture of an arbitrary original widget runtime. **Status:** research prototype and small user study.

<!-- Origin: precedents.md lines 123–123 -->

Primary sources: [project](https://hci.stanford.edu/research/mashups/), [paper](https://hci.stanford.edu/cstr/reports/2007-09.pdf)

<!-- Origin: precedents.md lines 127–127 -->

### WebMakeup — visual web augmentation

<!-- Origin: precedents.md lines 129–129 -->

This research tool lets users move/remove nodes and add material from different web pages. Its thesis explicitly addresses locator fragility after website updates and introduces alternative locators.

<!-- Origin: precedents.md lines 131–131 -->

**Depth:** DOM-based augmentation; web-only. **Status:** implemented Chrome-extension research tool documented in thesis; current distribution was not verified.

<!-- Origin: precedents.md lines 133–133 -->

Primary source: [author's university thesis record](https://ekoizpen-zientifikoa.ehu.eus/documentos/5ecb7f7c2999521315203e3b)

<!-- Origin: precedents.md lines 135–135 -->

### Wildcard — spreadsheet-driven browser customization

<!-- Origin: precedents.md lines 137–137 -->

Wildcard exposes a simplified table view of supported web-app data for user modification, annotations, and spreadsheet-style transformations. Its public MIT-licensed source is explicitly pre-release; the README warns that some advertised features, including filtering and formulas, were not yet ported to its current branch.

<!-- Origin: precedents.md lines 139–139 -->

**Depth:** extracted/adapter-mediated structured data with page customization. **Existing apps:** yes, supported websites; not any site without adaptation. **Status:** genuine public research code with explicit incompleteness.

<!-- Origin: precedents.md lines 141–141 -->

Primary sources: [project paper](https://www.geoffreylitt.com/wildcard/salon2020/), [repository](https://github.com/geoffreylitt/wildcard)

<!-- Origin: precedents.md lines 145–145 -->

### TabFS — live browser state as ordinary files

<!-- Origin: precedents.md lines 147–147 -->

TabFS connects a browser extension to a FUSE filesystem, making browser state accessible through ordinary filesystem tools. The native component forwards requests to the extension. It has public GPL-3.0 source.

<!-- Origin: precedents.md lines 149–149 -->

**Depth:** browser-state/control interoperability, not a visual fragment host. **Existing apps:** browser tabs, within extension capabilities.

<!-- Origin: precedents.md lines 151–151 -->

Primary sources: [author's explanation](https://omar.website/tabfs/), [source](https://github.com/osnr/TabFS)

<!-- Origin: precedents.md lines 153–153 -->

## 4. Native component and hyperdocument ancestry

<!-- Origin: precedents.md lines 155–155 -->

### OLE / COM compound documents

<!-- Origin: precedents.md lines 157–157 -->

OLE is deployed technology for placing objects made by different applications into one document and activating the original editing facilities in context. Microsoft's documentation makes the required COM, persistence, storage, and data-transfer interfaces explicit.

<!-- Origin: precedents.md lines 159–159 -->

**Depth:** real embedded/linked objects and original editors, often considerably deeper than pixels. **Existing apps:** participating applications, not arbitrary unmodified software. **Status:** documented platform technology, not just a concept.

<!-- Origin: precedents.md lines 161–161 -->

Primary source: [Microsoft compound documents](https://learn.microsoft.com/en-us/windows/win32/com/compound-documents)

<!-- Origin: precedents.md lines 165–165 -->

### OpenDoc

<!-- Origin: precedents.md lines 167–167 -->

Apple's historical developer material describes part editors and viewers whose functionality appears inside compound documents. The documented model supplies editing capabilities through parts.

<!-- Origin: precedents.md lines 169–169 -->

**Depth:** participant components; **status:** historical implemented platform, not a current adoption recommendation. It did not automatically decompose all legacy applications.

<!-- Origin: precedents.md lines 171–171 -->

Primary source: [Apple develop article, preserved](https://preserve.mactech.com/articles/develop/issue_22/opendoc.html)

<!-- Origin: precedents.md lines 173–173 -->

### KDE KParts

<!-- Origin: precedents.md lines 175–175 -->

KParts reuses GUI components, with viewer/editor parts and optional browser/text/scripting interfaces. This is another concrete, inspectable native component ecosystem.

<!-- Origin: precedents.md lines 177–177 -->

**Depth:** participating GUI components. **Constraint:** software must expose a KPart or compatible component; it is not arbitrary external window surgery.

<!-- Origin: precedents.md lines 179–179 -->

Primary sources: [KDE tutorial](https://techbase.kde.org/Development/Tutorials/Using_KParts), [interfaces](https://api.kde.org/legacy/4.14-api/kdelibs-apidocs/kparts/html/annotated.html)

<!-- Origin: precedents.md lines 181–181 -->

### Engelbart's Open Hyperdocument System

<!-- Origin: precedents.md lines 183–183 -->

OHS frames fine-grained addressability, transclusion, alternate views, provenance, and cross-vendor collaboration as foundational infrastructure. It is not merely an infinite canvas. The institute distinguishes the broad framework and evolving prototypes from a fully realized universal deployment.

<!-- Origin: precedents.md lines 185–185 -->

Primary source: [OHS overview](https://dougengelbart.org/content/view/156/)

<!-- Origin: precedents.md lines 189–189 -->

### Plan 9 plumbing

<!-- Origin: precedents.md lines 191–191 -->

The plumber routes context-sensitive messages to appropriate tools, such as taking a compiler error to the matching file/line. It demonstrates cross-tool relationships that are neither a monolithic app nor a dashboard.

<!-- Origin: precedents.md lines 193–193 -->

Primary source: [Plan 9 plumbing examples](https://9p.io/sources/wiki/d/18.hist)

<!-- Origin: precedents.md lines 197–197 -->

## 5. Malleable substrates: deep composition, with an adoption boundary

<!-- Origin: precedents.md lines 199–199 -->

### Webstrates / Codestrates / Varv

<!-- Origin: precedents.md lines 201–201 -->

Webstrates persists and synchronizes DOM changes, including code, and supports transclusion. Codestrates supplies authoring/execution tools. Varv expresses interactive software through declarative concepts, schemas, and actions; extensions can add or override behavior incrementally. Its demonstrations include recombining game rules and adapting views.

<!-- Origin: precedents.md lines 203–203 -->

These projects provide real public source, and Webstrates organization activity includes updates into 2026. Varv's documentation distinguishes its normal Webstrates workflow from an early Electron proof of concept that is not offered as a release.

<!-- Origin: precedents.md lines 205–205 -->

**Depth:** high inside the model, including state and behavior. **Boundary:** the source must participate in this environment or be adapted; a hosted third-party SaaS does not become a webstrate merely because it uses a DOM.

<!-- Origin: precedents.md lines 207–207 -->

Primary sources: [Webstrates publications](https://webstrates.net/project/publications/), [Varv paper](https://vis.mit.edu/pubs/varv/), [Varv usage/status](https://varv.projects.cavi.au.dk/docs/usage/), [public projects](https://github.com/Webstrates)

<!-- Origin: precedents.md lines 211–211 -->

### Potluck and Embark

<!-- Origin: precedents.md lines 213–213 -->

Potluck gradually enriches text documents with computations and interactive behavior. Embark uses more structured outlines, typed mentions such as places/dates, rich maps/calendars, and computations for travel planning. Both are explicitly research prototypes.

<!-- Origin: precedents.md lines 215–215 -->

**Depth:** meaningful relationships and computations in a new authoring substrate. **Existing apps:** integrations/data sources, not arbitrary native app surfaces.

<!-- Origin: precedents.md lines 217–217 -->

Primary sources: [Potluck](https://www.inkandswitch.com/potluck/), [Embark](https://www.inkandswitch.com/embark/)

<!-- Origin: precedents.md lines 219–219 -->

### Engraft — UIST 2023

<!-- Origin: precedents.md lines 221–221 -->

Engraft offers an API for recursively embedding live, rich programming tools in other tools and hosts. It is public source, with a paper and demonstrations; documentation is described as limited.

<!-- Origin: precedents.md lines 223–223 -->

**Depth:** composable programmable components, not magical extraction from opaque apps.

<!-- Origin: precedents.md lines 225–225 -->

Primary sources: [project](https://engraft.dev/), [paper](https://engraft.dev/engraft-uist-2023.pdf)

<!-- Origin: precedents.md lines 227–227 -->

### Dash — Brown hypermedia system

<!-- Origin: precedents.md lines 229–229 -->

Brown's Dash is a browser-based collaborative environment for spatial collections, fine-grained links, metadata, multimedia documents, and multiple composable views. Its documentation describes an online service and classroom use.

<!-- Origin: precedents.md lines 231–231 -->

**Depth:** common document/relationship model. **Boundary:** reuses and organizes heterogeneous media through Dash's implementation; it is not evidence of arbitrary third-party application behavior being transplanted. Do not confuse it with Plotly Dash.

<!-- Origin: precedents.md lines 233–233 -->

Primary sources: [Dash documentation](https://brown-dash.github.io/Dash-Documentation/about/), [architecture by its developers](https://hackmd.io/o7giQJUmTQ6mtyL3id0EVA)

<!-- Origin: precedents.md lines 235–235 -->

### Polyphony — Raffaillac and Huot, EICS 2019

<!-- Origin: precedents.md lines 237–237 -->

The relevant Polyphony is an experimental GUI toolkit using Entity–Component–System design: entities carry data components, and reusable systems select and act on them. This loosens the traditional widget/object hierarchy.

<!-- Origin: precedents.md lines 239–239 -->

**Depth:** internal toolkit architecture. **Boundary:** new or adapted software must be built in that model; it does not extract arbitrary GUI components. Paper and author thesis verified; a current supported release was not verified. There are numerous unrelated tools named Polyphony.

<!-- Origin: precedents.md lines 241–241 -->

Primary source: [author's thesis](https://traffaillac.github.io/content/manuscrit.pdf); paper DOI: https://doi.org/10.1145/3331150

<!-- Origin: precedents.md lines 243–243 -->

## 6. The less visible issue: preserving meaning across tool boundaries

<!-- Origin: precedents.md lines 245–245 -->

### Cambria — bidirectional edit lenses

<!-- Origin: precedents.md lines 247–247 -->

Cambria implements schema transformations as composable bidirectional lenses, with an experimental collaborative issue tracker. Its authors explicitly demonstrate incompatibility tradeoffs: a single-assignee model and a multi-assignee model cannot always preserve every desirable property simultaneously. Their design retains schema-tagged original writes and translates for readers rather than forcing one universal schema.

<!-- Origin: precedents.md lines 249–249 -->

**Status:** implemented TypeScript research library and prototype.

<!-- Origin: precedents.md lines 251–251 -->

Primary source: [Cambria](https://www.inkandswitch.com/cambria/)

<!-- Origin: precedents.md lines 253–253 -->

### In-the-wild substrates — Trividic, 2025

<!-- Origin: precedents.md lines 255–255 -->

This position paper explicitly argues for making the existing heterogeneous software ecology more interoperable instead of imposing an unfamiliar replacement workflow. It grounds this in publishing: Propage supports multiformat outputs and linked views; OutDesign helps route InDesign's IDML through Pandoc. The work includes contributions to WeasyPrint and Pandoc. The paper distinguishes practical conversion from the much harder goal of full bidirectional synchronization.

<!-- Origin: precedents.md lines 257–257 -->

Primary source: [Synchronising Content Across Formats In-the-wild](https://software-substrates.github.io/proceedings/2025/statements/Paper%2010%20-%20Yann%20Trividic%20-%20Synchronising%20Content%20Across%20Formats%20In-the-wild.pdf)

<!-- Origin: precedents.md lines 261–261 -->

## 8. Names, uncertainty, and false friends

<!-- Origin: precedents.md lines 302–302 -->

- **Metamuse** resolves in this research lineage to Muse's tools-for-thought podcast, not a verified composition platform. There are unrelated marketing, algorithm-generation, and other products using similar names. Do not list “Metamuse” as a technical predecessor without identifying the intended referent. [Muse's own episode page](https://allume.com/podcast/1-tool-switching/)
- **Polyphony** here means Raffaillac/Huot's ECS GUI toolkit, not a synthesis compiler or music application
- **Dash** here means Brown's hypermedia environment, not Plotly
- **Prefab** here means Dixon/Fogarty's pixel reverse-engineering toolkit, not current generative-UI libraries bearing that name
- **WinCuts** here means the Microsoft Research window-region system, not the unrelated shortcut utility
- “Pass-through mods” now has concrete likely matches, but the phrase alone does not specify which rendering/state strategy is used

<!-- Origin: precedents.md lines 304–309 -->
