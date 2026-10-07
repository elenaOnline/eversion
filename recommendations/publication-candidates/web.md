> Publication candidate: original research and recommendations, preserved for later comparison. Personal-context redactions are marked or generalized; this is not the byte-identical original. Withhold from both independent designers until both first drafts are frozen.

# Live composition of arbitrary web-app slices

Research memo, 7 October 2026. Scope: a desktop compositor can use existing websites without their source repositories or a vendor-provided plugin. Public primary documents, source/header files, and research papers were inspected; no software was installed, no user machine used, and no authenticated app compatibility was tested. Distinguish documented API capabilities, author-reported demos, and architectural inference below.

## Bottom line

A serious implementation is feasible if “slice” means **a persistent, interactive projection of part of a still-running application**. Keep the original execution context, page, origin, and session alive; crop/composite its rendered output and route input back. This can support canvas/WebGL apps as well as DOM apps. A desktop host controls the browser embedding/compositing layer, so ordinary iframe restrictions do not rule it out.

There is no general platform primitive that turns an arbitrary DOM selection into an **independent, semantically complete, portable component** retaining all its state and behavior. DOM ancestry does not establish a boundary around closures, framework state, workers, origin-bound storage, and backend transactions. One can engineer per-app extraction/adaptation or carry the entire original runtime along; the latter is virtualization/projection, even when it looks like a native widget.

Recommended foundation: a **browser-backed surface compositor with semantic adapters**, not a DOM copier. Start with supported embedding/capture APIs; consider a browser-engine modification only for the extra guarantees those APIs cannot provide. Do not implement the security model as “disable web security everywhere.”

## 1. Five different things people call a live embedded slice

1. **Live pixels, read-only.** A crop of ongoing rendering. Updates can be real time but buttons in the destination do nothing unless separately routed. Screenshots refreshed periodically are weaker again.
2. **Interactive live projection.** The real app remains running; the destination forwards pointer, keyboard, focus, and other input. App logic remains authentic, subject to correct routing. This is the strongest broadly applicable no-cooperation model.
3. **DOM mirror with remote actions.** A copied/mirrored DOM gives native text selection and destination layout, while actions go back to the original page. This is a remote UI protocol that someone must implement, not automatic transplantation. Full UI fidelity and action identity are major problems.
4. **In-place augmentation.** An extension/userscript alters layout and inserts tools inside the original app. Often powerful; the app remains the host and its document/runtime remain authoritative.
5. **Autonomous extracted component.** The original page/runtime can die and the component still works correctly elsewhere. Requires a complete dependency and behavior boundary, app-specific reverse engineering, or a deliberately composable substrate. A universal “pick any div” promise does not follow from browser APIs.

A product should label which of these it offers. “Live,” “embed,” “portal,” and “native” are not enough to determine behavior.

## 2. Architecture spectrum and honest capability envelope

### A. Plain iframe, optionally cropped and translated

An iframe keeps a complete page running. Positioning a larger iframe behind a smaller clipping viewport can display a region while browser-native input still reaches it. It is an economical implementation for permitted embeddings, including cooperating internal apps.

Constraints are specific, not interchangeable:

- `frame-ancestors` and X-Frame-Options govern whether a page may be framed. A frame can be disallowed even when network requests succeed. [MDN frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors)
- Same-origin policy normally prevents the parent from inspecting/manipulating a cross-origin frame. Merely being allowed to display a page does not give DOM access. Conversely, CORS governs readable cross-origin responses; setting CORS does not by itself authorize frame embedding or parent DOM access. [MDN same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)
- Cookie availability can change in a cross-site embedded context. SameSite and third-party storage policies are independent of visual embedding; a publicly readable page working in a frame proves little about its logged-in behavior. [MDN third-party cookies](https://developer.mozilla.org/en-US/docs/Web/Privacy/Guides/Third-party_cookies)

Inference: robust crop coordinates typically need cooperation or a privileged extension in each relevant frame. A static offset can work but drifts after responsive layout, banners, fonts, navigation, or scroll. Framing is not the universal core of the desktop design, but is still a good low-privilege route when available.

### B. Browser extension: preserve the original tab and instrument it

Content scripts share the page DOM while running in an isolated JavaScript world by default. They can measure, observe, restyle, and inject UI. They do not automatically see the page's JS globals. MAIN-world execution exists, but exposes the bridge to the page and is subject to page CSP. Frame-specific injection and host matching matter. [Chrome content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts)

An extension background/service-worker context can issue cross-origin requests with appropriate host permissions; content-script fetches remain subject to the origin's restrictions. A privileged background fetch is therefore a useful asset/data channel, not a general way to transplant a running app. [Chrome network requests](https://developer.chrome.com/docs/extensions/develop/concepts/network-requests)

`activeTab` is a least-privilege starting point: user invocation grants temporary access, revoked on cross-origin navigation or tab closure; restricted browser pages remain unavailable. Persistent multi-site boards need a consciously broader access model, not an assumption that one initial click authorizes every future origin. [Chrome activeTab](https://developer.chrome.com/docs/extensions/develop/concepts/activeTab)

Practical architecture (inference): selection overlay → semantic/DOM anchor → mutation/resize/scroll observation → tab surface capture → compositor tile → input bridge. Keep each source app in its own origin and broker only intended messages. Extension-only distribution inherits browser API and store-policy restrictions, whereas a trusted native/browser host can use a richer composition layer.

Closed shadow roots are **not** an absolute extension barrier: Chrome explicitly provides `chrome.dom.openOrClosedShadowRoot()`. This makes inspection possible with the relevant extension context, but does not make custom-element behavior portable. [Chrome DOM API](https://developer.chrome.com/docs/extensions/reference/api/dom)

### C. Electron native embedding and CEF

Electron currently documents three embedding options: iframe, webview, and WebContentsView. It discourages dependence on the webview tag because of architectural instability. WebContentsViews are main-process-controlled native views rather than DOM children, allowing pages to be arranged/layered in one window. [Electron Web Embeds](https://www.electronjs.org/docs/latest/tutorial/web-embeds)

A WebContentsView can adopt an existing WebContents, but one WebContents may be presented in only one WebContentsView at a time. Thus “show five crops of one already-running tab” is not solved by attaching that same WebContents to five native views. Use a shared captured surface, separate instances, or custom compositor support. [WebContentsView API](https://www.electronjs.org/docs/latest/api/web-contents-view)

Electron offers offscreen rendering as a bitmap or GPU shared texture. Its GPU path supports WebGL and CSS 3D; shared textures avoid the GPU→CPU bitmap copy but require native integration. Dirty-region updates and rendering controls are documented. This is a supported route to compositing pages into arbitrary destinations rather than pretending they are DOM children. [Electron offscreen rendering](https://www.electronjs.org/docs/latest/tutorial/offscreen-rendering)

The webContents API exposes input dispatch and frame subscriptions. A particularly important documented caveat is that `sendInputEvent()` needs the containing BrowserWindow focused. Do not assume a hidden window plus a few mouse events provides universal background interactivity. Focus, text input, and popup behavior must be designed/tested. [Electron webContents](https://www.electronjs.org/docs/latest/api/web-contents#contentssendinputeventinputevent)

CEF is an alternative for a native compositor. Its OSR model delivers rendering to the host and accepts host-supplied input. **Documentation trap:** an older General Usage page still says OSR does not support accelerated compositing; the current source header has `OnAcceleratedPaint` with Windows shared texture handles, macOS IOSurface, and Linux native-buffer planes. The current source is stronger evidence than the stale sentence. Texture lifetime/copy requirements are explicit in the header. [CEF current render handler](https://github.com/chromiumembedded/cef/blob/master/include/cef_render_handler.h), [CEF tutorial](https://chromiumembedded.github.io/cef/tutorial), [older General Usage page](https://chromiumembedded.github.io/cef/general_usage)

Inference: loading the original URL as a native-hosted top-level browser context preserves original web origin and avoids the particular “external page frames me” relationship. That is why iframe-only prohibitions are not a platform-level impossibility result. However, an embedded engine has its own browser profile, login lifecycle, supported features, and vendor-acceptance questions. It does not silently become the user's existing Chrome tab.

### D. CDP as an instrumentation/control channel

Chrome's debugger extension API exposes a restricted set of DevTools Protocol domains and requires the debugger permission. It can route commands/events to targets and child sessions. It is a powerful bridge, not an unrestricted promise that all Chrome functionality or security surfaces are accessible. [Chrome debugger API](https://developer.chrome.com/docs/extensions/reference/api/debugger)

The protocol exposes page capture/screencast, DOM inspection, and input domains. For an existing authorized session it can supply semantic anchors and remote interaction; encoded screencast transport is distinct from a zero-copy local GPU embedding interface. APIs marked experimental should be version-pinned and tested. [CDP Page](https://chromedevtools.github.io/devtools-protocol/tot/Page/), [CDP DOM](https://chromedevtools.github.io/devtools-protocol/tot/DOM/), [CDP Input](https://chromedevtools.github.io/devtools-protocol/tot/Input/)

Chrome 136 changed remote-debugging startup flags: default-profile debugging via `--remote-debugging-port` or pipe is no longer accepted; a nonstandard user-data directory is required. Chrome for Testing retains the automation-oriented behavior. This specifically limits the naive “launch your normal Chrome with a debugging flag and inherit all logins” plan. It is not a claim that user-authorized extension debugging can never inspect an existing tab. [Chrome announcement](https://developer.chrome.com/blog/remote-debugging-port)

### E. Screen, region, and element capture

Region Capture is a geometric crop; Element Capture selects a DOM subtree's rendering and can exclude unrelated occluders. Chrome's Element Capture documentation currently limits it to self-capture, requires an eligible cohesive target/stacking context, and says frames stop while a target is ineligible. Restriction does not make hidden `display:none` content render. These are video APIs, not portable interactive component APIs. [Chrome Element Capture](https://developer.chrome.com/docs/web-platform/element-capture)

Chrome extension `tabCapture` provides tab media capture after user invocation. It is useful for an extension-to-desktop or browser compositor, but must be combined with an independent action/input path. [Chrome tabCapture](https://developer.chrome.com/docs/extensions/reference/api/tabCapture)

Inference: capture API naming should not obscure the separate design problems of source identity, source liveness, selection, hit testing, text/IME, browser popups, source coordinate transforms, and action confirmation. A browser fork could improve the capture/compositor layer; it would not automatically derive semantic app boundaries.

### F. Reverse proxy and JavaScript virtualization

A reverse proxy can rewrite URLs, headers, cookies, HTML/CSS, and script-visible APIs so an app appears under a controlled origin. Simply removing frame headers is far short of a modern app virtualizer: runtime-generated URLs, WebSockets, service workers, navigation, storage, and origin expectations all matter.

**Webfuse** is a concrete current product in this category. Its infrastructure docs explicitly describe all session traffic passing through its proxy, response/URL rewriting, and a controlled JS sandbox intercepting execution points and Web API interactions; rendering is local. That is more substantial than a CSS-injection product. The same page's marketing says no remote processing/exposure, which should not be interpreted literally given its explicit traffic-proxy architecture. Local rendering does not mean the proxy cannot see session traffic. [Webfuse infrastructure](https://dev.webfuse.com/infrastructure/)

Its extension docs specify popup/background/content-script-like modules installed into a platform Space, with page-window and persistent session scope. It supports custom UI on/in original app UIs but excludes browser-wide APIs such as bookmarks. This is a relevant implementation model for no end-user extension installation, while still depending on platform code and deployment. [Webfuse Extensions](https://dev.webfuse.com/extensions/)

Vendor claims of “any app,” “all behavior,” or zero latency were not independently established. An engineering plan should require a compatibility corpus and inspect auth/worker/device/API edge cases rather than use those slogans as proofs. The vendor itself explains `location`/CookieStore and hardcoded-resource problems in naive proxying. [Webfuse embedding explanation](https://www.webfuse.com/blog/how-to-embed-any-website)

### G. Remote-browser-backed embed

Run the actual browser elsewhere and stream its output/input channel into the desktop surface. This preserves more source behavior than a DOM mirror and avoids remote-page framing because the remote page is not the iframe's direct document. Costs are latency, transport, remote-session trust, device features, credentials, accessibility integration, and persistence.

**BrowserBox** documents a remote browser isolation product with embedding/automation. Its current repository says it is proprietary commercial software distributed as binaries, and says legacy source was removed in March 2026. Do not describe today's BrowserBox core as open source because older forks/search results do. Its performance/security claims were not benchmarked here. [Current BrowserBox repository](https://github.com/BrowserBox/BrowserBox)

**Hyper-Frame**, the BrowserBox custom element, is separately published with AGPL-3.0-or-later. Its documented API requires a BrowserBox login link; exposes tabs, navigation, selectors, evaluation, capture, policy and lifecycle; and has explicit embedder-origin controls. This is an iframe wrapping a BrowserBox session/API, not a browser-standard escape hatch that makes the target server's page directly embeddable. Public source tree was found, but the individual JS file failed to fetch in this research run. [Hyper-Frame repository/API](https://github.com/BrowserBox/hyper-frame), [current embedding guide](https://docs.browserbox.io/embedding/)

Browserless's self-hosted Live Debugger also explicitly provides an interactive visual screencast. This confirms availability of useful remote-browser interaction plumbing, not arbitrary slice extraction as a complete product. [Browserless Live Debugger](https://docs.browserless.io/enterprise/live-debugger)

## 3. Why literal DOM transplantation is not a general solution

`cloneNode()` omits addEventListener/property listeners and canvas painted contents. `importNode()` copies into a new document/custom-element registry; it is not a runtime-state serializer. Even perfect CSS capture cannot supply missing application behavior. [MDN cloneNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)

Moving the original node instead of cloning may preserve its identity/listeners, but is still risky: document-level event delegation, DOM ancestry, styles, layout measurements, owner document, focus, observers, framework reconciliation, and custom-element lifecycle may change. The important distinction is between a DOM object and the application's effective component boundary. This is architectural analysis, not a claim that every node move fails.

React itself warns that adding/removing/modifying nodes it manages can cause inconsistency or crashes and provides a concrete manual-removal example. Reactive apps can recreate the subtree or reapply styles after your mutation. An adapter can work with that behavior; a mutation observer that constantly “fights React” is not equivalent to correctness. [React DOM manipulation guidance](https://react.dev/learn/manipulating-the-dom-with-refs)

React portals are different: app code renders JSX through a portal while preserving React ancestry/context and React event propagation. They require access to the executing component/tree/React integration point; an arbitrary external DOM selection does not become a React portal by being moved. [React createPortal](https://react.dev/reference/react-dom/createPortal)

A mirror also needs non-HTML state. rrweb's serialization describes separately recording input state, rewriting URLs, inlining styles, assigning node IDs, and de-scripting replay. Its sandbox deliberately avoids rerunning original scripts. This is excellent evidence for what a replay system does, and why replay is not an independently functioning copy of the app. [rrweb serialization](https://github.com/rrweb-io/rrweb/blob/main/docs/serialization.md), [rrweb sandbox](https://github.com/rrweb-io/rrweb/blob/main/docs/sandbox.md)

A current static-extraction counterexample is **DOM Capture**: it offers style-complete HTML and shadow-root support, but its own README explicitly says JavaScript behavior does not travel. It also discusses session-signed iframe sources. Useful capture tooling should not be marketed as working-component extraction. [DOM Capture](https://github.com/schappim/dom-capture)

## 4. Research precedents: what was actually demonstrated

### Fusion, UIST 2018: closest direct precedent

Xiong Zhang and Philip Guo's Fusion is a Chrome-extension UI-mashup prototype: select from unmodified pages, iframe-transclude widgets, and connect them with JavaScript glue. Authors report seven case studies, each under 15 glue-code lines, including Python Tutor inside tutorials, code/documentation lookup, LaTeX preview, data-science tooling, and collaboration. This is author-reported demonstration evidence, not this memo's tested compatibility result.

Crucial limitations: literal CSS selectors; iframe restrictions; finding actionable DOM/events in complex apps; no generic deeper backend integration. Their cross-domain setup used disabled same-origin checks for prototyping or a same-origin proxy alternative. The paper calls it medium-fidelity prototyping rather than production infrastructure. It establishes that useful no-source UI composition is real while illustrating what a product must harden. [Author-hosted full paper, especially pp. 6–9](https://pg.ucsd.edu/publications/Fusion-opportunistic-web-prototyping-UI-mashups_UIST-2018.pdf)

### C3W / Clip, Connect, Clone

Historical research implemented visual cells as portals to web pages, with recorded navigation and HTML/XPath-style addressing. Input/result cells were connected by spreadsheet-like formulas; cloning let users compare alternatives. This is directly relevant to a semantic broker between app slices. The paper describes a specialized browser/PlexWare/IntelligentPad implementation, not a currently maintained universal modern-SPA library. Primary author paper available through ResearchGate: [Clip, connect, clone](https://www.researchgate.net/publication/220877248_Clip_connect_clone_Combining_application_elements_to_build_custom_interfaces_for_information_access), [short C3W paper](https://www.researchgate.net/publication/221023398_C3W_clipping_connecting_and_cloning_for_the_web)

### WebMakeup

A university thesis/project record describes a Chrome extension for selecting, moving, removing, and adding nodes from other pages; it explicitly identifies site updates/locator robustness as a difficulty. Strong conceptual precedent for user-authored augmentation, weak evidence for modern arbitrary authenticated-SPA portability. [University thesis record](https://www.ehu.eus/eu/web/doktoregoa/ingeniaritza-informatikoa-doktoregoa/defendatutako-tesiak)

### Webstrates and MyWebstrates

Webstrates synchronizes persistent DOM changes across clients and composes webstrates via transclusion. It is a powerful purpose-designed substrate; it does not prove arbitrary proprietary applications can be extracted with equivalent semantics. Its original paper says state not represented in the DOM is not shared. MyWebstrates (UIST 2024) advances a local-first version of the substrate. [Webstrates documentation](https://webstrates.github.io/), [original author paper](https://pure.au.dk/ws/files/91047333/webstrates.pdf), [MyWebstrates author paper](https://cs.au.dk/~clemens/files/MyWebstrates-UIST2024.pdf)

### d.mix and Bricolage: adjacent, not equivalent

Stanford's d.mix samples annotated sites into API calls: useful site-to-service translation but depends on a mapping to services/APIs. Bricolage transfers content into another page's style/layout using learned mappings. Neither is evidence of universally preserving hidden runtime behavior of arbitrary selected controls. [d.mix project](https://hci.stanford.edu/research/mashups/), [Bricolage author-hosted paper](https://hci.stanford.edu/publications/2011/Bricolage/Bricolage-CHI2011.pdf)

## 5. Failure cases and which layer can address them

The following is architectural inference grounded in the mechanisms above. “Hard” is not used as a synonym for impossible.

| Case | Projection/compositor | DOM extraction or mirror | What remains genuinely constrained |
|---|---|---|---|
| React/Vue/other SPA re-render | Preserve page runtime; reacquire anchor if nodes change | Copied state/handlers become stale; moving nodes can conflict with renderer | No stable vendor-independent semantic IDs guaranteed |
| Canvas/WebGL app | Pixels and source input can preserve functionality | DOM may expose one canvas, not buttons/text/shapes | Cannot derive unavailable semantic structure by reading DOM; vision is inference |
| Closed shadow DOM | Rendering unaffected; privileged inspection may exist | Ordinary page JS cannot use `.shadowRoot`; extension API can help | Closed shadow root is not an absolute trusted-browser barrier |
| Cross-origin nested iframe | Capture includes rendering; source input can reach it | Parent cannot freely inspect; instrument authorized frames independently | Origin separation remains unless privileged broker mediates |
| Authenticated app | Original context/session is best | Proxy/moved-origin flow may break cookies, storage, redirects | Server authorization, origin-bound auth, device and enterprise policies remain |
| Modal, tooltip, popover outside selected subtree | Broaden crop or composite transient surface | Subtree mirror can omit it entirely | A visual “component” is not necessarily a DOM subtree |
| File picker, permission UI, WebAuthn | Need host/native handling and explicit identity | Not serializable ordinary HTML | User/OS/browser security interaction cannot be assumed away |
| Several slices of one tab | Multiple crops can share one texture | Multiple independent DOM copies diverge | Original page has one current route/focus/scroll state; independent views need more state |
| Virtualized/offscreen content | Source viewport must cause needed content to render | Absent DOM has nothing to copy | A crop cannot reveal data/content the app never instantiated |
| Drag/drop, IME, clipboard, selection | Full input protocol and coordinate/focus handling | Generic click forwarding insufficient | OS/browser trust and permission semantics still apply |
| Background tab suspension/crash | Keep-alive/resource scheduling, restore source | Mirrored view can falsely look alive | Cannot show live data while authoritative source is stopped; expose stale state |
| Unexposed backend operation | Can perform real available UI actions | No newly created backend capability | Client-side composition cannot manufacture server-side authority |

For multiple source apps, there is no generic global undo or transaction. If action A succeeds and B fails, compositor-side rollback cannot assume A is reversible. Cross-app automation must preserve operation identity, expected preconditions, confirmation, and recovery state.

## 6. Authentication/security boundaries that survive unlimited engineering effort

**App-origin authentication is not just a URL string.** WebAuthn credentials are scoped to a relying-party ID and browser-checked origin constraints. Rewriting a website to a proxy origin does not automatically make its existing passkeys usable there. Related-origin requests are defined, but require the relying party's permitted-origin machinery; they are not unilateral arbitrary-proxy authorization. [WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/)

**Provider acceptance is separate from rendering ability.** Google's OAuth policy disallows authorization in developer-controlled embedded user agents and requires verifiable connection identity. An Electron browser can render HTML perfectly and still not be an acceptable login surface. A compliant external-browser login route may work when the application supports the redirect/token handoff; a third-party compositor cannot presume it can retrofit that into every SaaS service. [Google OAuth policies](https://developers.google.com/identity/protocols/oauth2/policies)

**Sharing a browser session is not cloning app state.** Electron supports named persistent/in-memory session partitions. Sharing a partition provides shared session storage facilities where appropriate; it does not duplicate a page's current JS heap, unsaved input, selected route, or in-flight task. Keep a source runtime alive if exact session continuity matters. [Electron session API](https://www.electronjs.org/docs/latest/api/session)

**Native trust does not imply renderer trust.** Electron's security guidance recommends sandboxing, context isolation, no Node integration for remote content, limited permissions/navigation/window creation, and validating IPC senders. These are essential because a composition app intentionally brings untrusted origins close to privileged desktop functionality. [Electron security guide](https://www.electronjs.org/docs/latest/tutorial/security)

Unlimited engineering can produce a custom browser, virtualized web runtime, and app-specific adapters. It cannot guarantee service availability, continued account access, a cooperative server response, cryptographic credentials unavailable to the user/runtime, or stable semantics of arbitrary changing apps. Do not confuse an intentional browser policy boundary with impossibility for a separately authorized trusted host; equally, do not confuse host control with permission to bypass service or user security.

## 7. Recommended implementation architecture

This section is a design recommendation, not a claim that a listed project ships the complete stack.

1. **Source runtime manager.** Each source is a real browser document/target at its original URL, owned or explicitly attached with user permission. Separate origin/session boundaries; avoid exporting raw credentials. Use native engine views or offscreen renderers for owned runtime sources, and an extension/CDP bridge for already-running browser sources where supported.
2. **Selection/anchor model.** Persist source identity, frame chain, semantic locator, visual reference, and fallback bounds. Track geometry separately from semantic identity. Fail closed on ambiguous rematching for consequential actions.
3. **Surface graph.** Tiles reference source output plus crop/transform/mask, not copied applications. Many tiles may refer to one source. Explicitly represent whether they share navigation, focus, and scroll state.
4. **Rendering paths.** Native WebContentsView for ordinary rectangular regions; offscreen shared textures for transformed/multiple crops; video transport for remote sources. Pixel path is a compatibility fallback for canvas/custom rendering.
5. **Input router.** Reverse transforms; route pointer capture, hover, wheel, key, IME, selection, drag/drop, clipboard, accessibility actions, focus, and popup/dialog requests. Stale-frame input must be detected. This is a substantial subsystem, not “onClick → click().”
6. **Transient-surface policy.** Detect dropdowns/modals/tooltips/navigation leaving the crop. Expand the tile, show a companion overlay, or switch to a full-app view. Always provide “open source in full context.”
7. **Semantic broker.** Optional adapters convert observable app state into typed events/actions. The broker mediates cross-app operations explicitly. UI side-by-side is spatial composition; “take selected customer in A and open related invoice in B” is semantic integration and needs a mapping.
8. **Safety and provenance chrome.** Show origin and account identity; make live/stale/read-only distinctions obvious; prevent a malicious tile from impersonating trusted app controls or silently gaining new powers. Store broad capture permissions separately from allowed action scope.
9. **Persistence.** Persist a recipe (source URL, intended state/anchors, adapter version, layout), with best-effort state rehydration. Do not promise exact arbitrary JS-heap resurrection after restart.
10. **Compatibility harness.** Test target workflows with source apps' actual login, nested frames, virtual lists, canvas, rich text, drag/drop, file flows, permissions, refresh, route change, interruption, offline/crash, and upgrades. Track per-operation confidence and regressions.

### When a browser fork becomes justified

A custom Chromium/CEF integration may be appropriate if the product must present one runtime in multiple composited places, reliably retain offscreen rendering, redirect hit testing into transformed subregions, include transient browser-owned surfaces, or expose a stable surface-selection protocol. These are browser/compositor engineering problems, and the user explicitly need not optimize for easy engineering.

A fork should preserve site isolation and origin-based security by default. Put user-authorized cross-origin operations in a capability broker. A general compositor fork is a coherent ambitious direction; globally disabling same-origin checks is an avoidable implementation shortcut with a very different trust model.

### What to prototype first to test the real thesis

- A single running authenticated SPA with two crops sharing exactly one source runtime, with an interaction in one immediately visible in the other.
- A canvas-based app tile with typing, drag, zoom, and popup handling.
- A DOM-backed search/list tile that survives rerender, scrolling, and navigation using an anchor recipe.
- Two apps connected by one explicit semantic action, including failure/retry without double submission.
- A login/permission/critical-action escape to full-context view with unmistakable origin/account identity.

Those tests are more diagnostic than a polished dashboard of static screenshots or permitted public iframes.

## 8. Evidence confidence and unanswered questions

- **High confidence:** browser-origin/frame constraints; DOM clone omissions; React reconciliation warning; extension isolated worlds; closed-shadow extension API; Electron one-view-per-WebContents rule; offscreen rendering/input primitives; CEF accelerated-render source interface; WebAuthn and Google OAuth constraints.
- **Medium confidence:** product-documented capabilities of Webfuse, BrowserBox/Hyper-Frame, Browserless; source/research-verified architecture, but no compatibility benchmark performed.
- **Historical demonstration evidence:** Fusion, C3W, WebMakeup. Useful proof of interaction techniques, not current availability or broad SaaS support.
- **Architectural inference:** proposed hybrid system and its trade-offs; credible path from documented primitives, not an already validated build.
- **Not established:** universal authenticated-app compatibility; exact performance with dozens of slices; zero-copy path on every OS/GPU; complete accessibility/IME integration; commodity-browser permission/store approval for every proposed extension feature; automatic semantic adapters robust to arbitrary vendor changes.

The strongest defensible claim is: **live surface composition without vendor plugins is buildable; extracting universal independent components with reliable semantics is a different and much stronger claim.**

## 9. Addendum: live-node movement, newer primitives, and generated adapters

### Same-document move is materially stronger than copying

These operations must not be conflated:

- **CSS-only rearrangement:** retains actual nodes and DOM ancestry. It can preserve most runtime and event structure but changes layout/geometry and may be overwritten by the app. This can be an excellent adapter strategy when a suitable subtree already exists.
- **Moving the actual node within its original document:** preserves the node object rather than creating a copy. `appendChild(existingNode)` explicitly moves it; directly attached listeners are not the cloning problem described earlier. Old and new positions cannot both contain that same node. [appendChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)
- **Atomic same-document move:** `moveBefore()` is a newer browser primitive. Chrome introduced it in 133; it avoids the implicit remove/reinsert and preserves state such as iframe loading, focus, animations, fullscreen, popovers and modal dialogs. It is directly relevant to a rearrangeable composition surface. [Chrome introduction](https://developer.chrome.com/blog/movebefore-api)
- **Cross-document adoption:** `adoptNode()` transfers the same node, removes it from the old document, and changes `ownerDocument`. This is not cloning, but it changes important document dependencies. It does not move every other object the component's callbacks close over, or grant access across origin boundaries. [adoptNode](https://developer.mozilla.org/en-US/docs/Web/API/Document/adoptNode)
- **Cross-document clone/import:** new nodes; event/paint/state omissions discussed above. This is the weakest option for preserving behavior.

`moveBefore()` only supports a move within one document and with compatible connected/disconnected status. It has a state-preserving custom-element callback mechanism, `connectedMoveCallback()`. Crucially, atomic DOM state preservation does not promise that a third-party framework tolerates the new tree shape, or that old ancestor-dependent CSS and JS still mean the same thing. [moveBefore reference](https://developer.mozilla.org/en-US/docs/Web/API/Element/moveBefore)

**Concrete event-path failure:** React 17 moved most delegated listeners from `document` to the React root container; React separately installs listeners on portal containers. Moving a React-managed node outside its original root without creating a real React portal can therefore sever the expected event path even while the node and its direct listeners survive. This is a version/framework integration issue, not evidence that movement cannot work. [React 17 event-delegation design](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html)

Native event paths also reflect DOM/shadow-tree relationships; outside observers do not see closed-shadow internal nodes in `composedPath()`. An adapter must reason about events at the right layer rather than assuming `target` or parent traversal always gives the meaningful control. [composedPath](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath)

### HTML-in-Canvas: promising primitive with intentionally narrower security scope

The current WICG HTML-in-Canvas explainer proposes browser-rendered DOM snapshots in 2D/WebGL/WebGPU plus geometry synchronization, accessibility, and hit-testing support. It is experimental/flagged; the API is evolving (older material uses `layoutsubtree`, current explainer uses `content="drawable"`/`drawable`). Security rules exclude information unavailable to author script, including cross-origin frame/content pixels and IME popups. It is therefore useful inspiration for element-level surfaces but **not** a generic cross-origin webpage capture/import permission. [WICG explainer](https://github.com/WICG/html-in-canvas), [Chrome origin-trial article](https://developer.chrome.com/blog/html-in-canvas-origin-trial)

### Where framework introspection and generated app hooks fit

A distributed component host can absolutely use framework-level introspection to discover the current logical component tree, current props/state, renderer commits, and likely action bindings, then generate app-specific hooks. This is a sound ambitious engineering path. The key distinction: “source-code independent” can mean no repository/build/vendor API required; it need not mean “has no site-specific understanding.”

Recommended adapter hierarchy (design inference):

1. Standard observable semantics: accessible roles/names, form behavior, DOM attributes and UI events.
2. Framework-aware adapters: detect/version renderer integration, map logical components to source nodes, observe commits, use supported or carefully versioned renderer hooks where available.
3. Generated app-specific adapters: identify workflow entities/actions, preconditions and confirmation signals; keep code per-site/version and validate against observed UI behavior.
4. Surface-only fallback: preserve full behavior with pixels/input when useful semantics cannot be reliably recovered.

Framework introspection can reveal how UI state is represented. It cannot guarantee that a field named `id` means a customer rather than an invoice, that invoking a callback directly reproduces a trusted user action, that a backend operation is reversible, or that an invisible server precondition has been satisfied. Generated adapters should expose uncertainty and own regression tests. Directly invoking an internal callback can also skip validation, batching, analytics, form defaults, focus, or app event sequencing; event/input-based execution is often safer until equivalence is demonstrated.

### Dedicated authentication contexts

For a host-owned engine, dedicate a persistent browser context per intended account/workspace identity, and let sources within that identity deliberately share it. Keep the app at its real origin where possible. Log in through a supported visible flow; show account identity on tiles. This avoids treating cookie export from existing browsers as the default integration mechanism.

A dedicated context solves repeat login/session organization but does not automatically pass an enterprise conditional-access policy or embedded-agent restriction. For apps that require the user's supported existing browser/device, prefer an authorized bridge that leaves the runtime there. A full source-view fallback belongs in the product's normal architecture, especially for auth, passkeys, permissions, and consequential confirmation screens.
