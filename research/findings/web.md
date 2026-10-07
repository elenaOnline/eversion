# Web: evidence and documented mechanisms

Editorial extraction, 2026-10-07. Source research was read-only; no listed software was installed, executed, or benchmarked. Documented APIs, author-reported demonstrations, source inspections, and untested compatibility are distinguished below. Source-level conclusions are limited to the investigated interfaces and versions. No implementation architecture is prescribed here.

1. **Live pixels, read-only.** A crop of ongoing rendering. Updates can be real time but buttons in the destination do nothing unless separately routed. Screenshots refreshed periodically are weaker again.
2. **Interactive live projection.** The real app remains running; the destination forwards pointer, keyboard, focus, and other input. App logic remains authentic, subject to correct routing.
3. **DOM mirror with remote actions.** A copied/mirrored DOM gives native text selection and destination layout, while actions go back to the original page. This is a remote UI protocol that someone must implement, not automatic transplantation. Full UI fidelity and action identity are major problems.
4. **In-place augmentation.** An extension/userscript alters layout and inserts tools inside the original app. Often powerful; the app remains the host and its document/runtime remain authoritative.
5. **Autonomous extracted component.** The original page/runtime can die and the component still works correctly elsewhere. Requires a complete dependency and behavior boundary, app-specific reverse engineering, or a deliberately composable substrate. A universal “pick any div” promise does not follow from browser APIs.

<!-- Origin: web.md lines 15–19 -->

## 2. Browser mechanisms and documented boundaries

<!-- Origin: web.md lines 23–23 -->

### A. Plain iframe, optionally cropped and translated

<!-- Origin: web.md lines 25–25 -->

An iframe keeps a complete page running. Positioning a larger iframe behind a smaller clipping viewport can display a region while browser-native input still reaches it.

<!-- Origin: web.md lines 27–27 -->

Constraints are specific, not interchangeable:

<!-- Origin: web.md lines 29–29 -->

- `frame-ancestors` and X-Frame-Options govern whether a page may be framed. A frame can be disallowed even when network requests succeed. [MDN frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors)
- Same-origin policy normally prevents the parent from inspecting/manipulating a cross-origin frame. Merely being allowed to display a page does not give DOM access. Conversely, CORS governs readable cross-origin responses; setting CORS does not by itself authorize frame embedding or parent DOM access. [MDN same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)
- Cookie availability can change in a cross-site embedded context. SameSite and third-party storage policies are independent of visual embedding; a publicly readable page working in a frame proves little about its logged-in behavior. [MDN third-party cookies](https://developer.mozilla.org/en-US/docs/Web/Privacy/Guides/Third-party_cookies)

<!-- Origin: web.md lines 31–33 -->

### B. Browser extension: preserve the original tab and instrument it

<!-- Origin: web.md lines 37–37 -->

Content scripts share the page DOM while running in an isolated JavaScript world by default. They can measure, observe, restyle, and inject UI. They do not automatically see the page's JS globals. MAIN-world execution exists, but exposes the bridge to the page and is subject to page CSP. Frame-specific injection and host matching matter. [Chrome content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts)

<!-- Origin: web.md lines 39–39 -->

An extension background/service-worker context can issue cross-origin requests with appropriate host permissions; content-script fetches remain subject to the origin's restrictions. [Chrome network requests](https://developer.chrome.com/docs/extensions/develop/concepts/network-requests)

<!-- Origin: web.md lines 41–41 -->

`activeTab` grants temporary access: user invocation grants temporary access, revoked on cross-origin navigation or tab closure; restricted browser pages remain unavailable. The documented temporary grant does not authorize all future origins. [Chrome activeTab](https://developer.chrome.com/docs/extensions/develop/concepts/activeTab)

<!-- Origin: web.md lines 43–43 -->

Closed shadow roots are **not** an absolute extension barrier: Chrome explicitly provides `chrome.dom.openOrClosedShadowRoot()`. This makes inspection possible with the relevant extension context, but does not make custom-element behavior portable. [Chrome DOM API](https://developer.chrome.com/docs/extensions/reference/api/dom)

<!-- Origin: web.md lines 47–47 -->

### C. Electron native embedding and CEF

<!-- Origin: web.md lines 49–49 -->

Electron currently documents three embedding options: iframe, webview, and WebContentsView. It discourages dependence on the webview tag because of architectural instability. WebContentsViews are main-process-controlled native views rather than DOM children, allowing pages to be arranged/layered in one window. [Electron Web Embeds](https://www.electronjs.org/docs/latest/tutorial/web-embeds)

<!-- Origin: web.md lines 51–51 -->

A WebContentsView can adopt an existing WebContents, but one WebContents may be presented in only one WebContentsView at a time. Thus “show five crops of one already-running tab” is not solved by attaching that same WebContents to five native views. [WebContentsView API](https://www.electronjs.org/docs/latest/api/web-contents-view)

<!-- Origin: web.md lines 53–53 -->

Electron offers offscreen rendering as a bitmap or GPU shared texture. Its GPU path supports WebGL and CSS 3D; shared textures avoid the GPU→CPU bitmap copy but require native integration. Dirty-region updates and rendering controls are documented. [Electron offscreen rendering](https://www.electronjs.org/docs/latest/tutorial/offscreen-rendering)

<!-- Origin: web.md lines 55–55 -->

The webContents API exposes input dispatch and frame subscriptions. A particularly important documented caveat is that `sendInputEvent()` needs the containing BrowserWindow focused. This does not establish universal background interactivity. [Electron webContents](https://www.electronjs.org/docs/latest/api/web-contents#contentssendinputeventinputevent)

<!-- Origin: web.md lines 57–57 -->

CEF is an alternative for a native compositor. Its OSR model delivers rendering to the host and accepts host-supplied input. **Documentation trap:** an older General Usage page still says OSR does not support accelerated compositing; the current source header has `OnAcceleratedPaint` with Windows shared texture handles, macOS IOSurface, and Linux native-buffer planes. The current source is stronger evidence than the stale sentence. Texture lifetime/copy requirements are explicit in the header. [CEF current render handler](https://github.com/chromiumembedded/cef/blob/master/include/cef_render_handler.h), [CEF tutorial](https://chromiumembedded.github.io/cef/tutorial), [older General Usage page](https://chromiumembedded.github.io/cef/general_usage)

<!-- Origin: web.md lines 59–59 -->

A native-hosted top-level browser context is not the same embedding relationship as an external page's iframe. It retains the loaded URL's original origin, but has its own profile, login lifecycle, engine features, and vendor acceptance conditions. It does not become an already-running Chrome tab automatically.

<!-- Origin: web.md lines 61–61 -->

### D. CDP as an instrumentation/control channel

<!-- Origin: web.md lines 63–63 -->

Chrome's debugger extension API exposes a restricted set of DevTools Protocol domains and requires the debugger permission. It can route commands/events to targets and child sessions. It is a powerful bridge, not an unrestricted promise that all Chrome functionality or security surfaces are accessible. [Chrome debugger API](https://developer.chrome.com/docs/extensions/reference/api/debugger)

<!-- Origin: web.md lines 65–65 -->

The protocol exposes page capture/screencast, DOM inspection, and input domains. For an existing authorized session it can supply semantic anchors and remote interaction; encoded screencast transport is distinct from a zero-copy local GPU embedding interface. Some APIs are marked experimental. [CDP Page](https://chromedevtools.github.io/devtools-protocol/tot/Page/), [CDP DOM](https://chromedevtools.github.io/devtools-protocol/tot/DOM/), [CDP Input](https://chromedevtools.github.io/devtools-protocol/tot/Input/)

<!-- Origin: web.md lines 67–67 -->

Chrome 136 changed remote-debugging startup flags: default-profile debugging via `--remote-debugging-port` or pipe is no longer accepted; a nonstandard user-data directory is required. Chrome for Testing retains the automation-oriented behavior. This specifically limits the naive “launch your normal Chrome with a debugging flag and inherit all logins” plan. It is not a claim that user-authorized extension debugging can never inspect an existing tab. [Chrome announcement](https://developer.chrome.com/blog/remote-debugging-port)

<!-- Origin: web.md lines 69–69 -->

### E. Screen, region, and element capture

<!-- Origin: web.md lines 71–71 -->

Region Capture is a geometric crop; Element Capture selects a DOM subtree's rendering and can exclude unrelated occluders. Chrome's Element Capture documentation currently limits it to self-capture, requires an eligible cohesive target/stacking context, and says frames stop while a target is ineligible. Restriction does not make hidden `display:none` content render. These are video APIs, not portable interactive component APIs. [Chrome Element Capture](https://developer.chrome.com/docs/web-platform/element-capture)

<!-- Origin: web.md lines 73–73 -->

Chrome extension `tabCapture` provides tab media capture after user invocation. It is useful for an extension-to-desktop or browser compositor, but must be combined with an independent action/input path. [Chrome tabCapture](https://developer.chrome.com/docs/extensions/reference/api/tabCapture)

<!-- Origin: web.md lines 75–75 -->

### F. Reverse proxy and JavaScript virtualization

<!-- Origin: web.md lines 79–79 -->

A reverse proxy can rewrite URLs, headers, cookies, HTML/CSS, and script-visible APIs so an app appears under a controlled origin. Simply removing frame headers is far short of a modern app virtualizer: runtime-generated URLs, WebSockets, service workers, navigation, storage, and origin expectations all matter.

<!-- Origin: web.md lines 81–81 -->

**Webfuse** is a concrete current product in this category. Its infrastructure docs explicitly describe all session traffic passing through its proxy, response/URL rewriting, and a controlled JS sandbox intercepting execution points and Web API interactions; rendering is local. The same page's marketing says no remote processing/exposure, which should not be interpreted literally given its explicit traffic-proxy architecture. Local rendering does not mean the proxy cannot see session traffic. [Webfuse infrastructure](https://dev.webfuse.com/infrastructure/)

<!-- Origin: web.md lines 83–83 -->

Its extension docs specify popup/background/content-script-like modules installed into a platform Space, with page-window and persistent session scope. It supports custom UI on/in original app UIs but excludes browser-wide APIs such as bookmarks. This deployment uses platform code rather than requiring an end-user browser extension. [Webfuse Extensions](https://dev.webfuse.com/extensions/)

<!-- Origin: web.md lines 85–85 -->

Vendor claims of “any app,” “all behavior,” or zero latency were not independently established. The vendor itself explains `location`/CookieStore and hardcoded-resource problems in naive proxying. [Webfuse embedding explanation](https://www.webfuse.com/blog/how-to-embed-any-website)

<!-- Origin: web.md lines 87–87 -->

### G. Remote-browser-backed embed

<!-- Origin: web.md lines 89–89 -->

Run the actual browser elsewhere and stream its output/input channel into the desktop surface. This preserves more source behavior than a DOM mirror and avoids remote-page framing because the remote page is not the iframe's direct document. Costs are latency, transport, remote-session trust, device features, credentials, accessibility integration, and persistence.

<!-- Origin: web.md lines 91–91 -->

**BrowserBox** documents a remote browser isolation product with embedding/automation. Its current repository says it is proprietary commercial software distributed as binaries, and says legacy source was removed in March 2026. Its performance/security claims were not benchmarked here. [Current BrowserBox repository](https://github.com/BrowserBox/BrowserBox)

<!-- Origin: web.md lines 93–93 -->

**Hyper-Frame**, the BrowserBox custom element, is separately published with AGPL-3.0-or-later. Its documented API requires a BrowserBox login link; exposes tabs, navigation, selectors, evaluation, capture, policy and lifecycle; and has explicit embedder-origin controls. This is an iframe wrapping a BrowserBox session/API, not a browser-standard escape hatch that makes the target server's page directly embeddable. Public source tree was found, but the individual JS file failed to fetch in this research run. [Hyper-Frame repository/API](https://github.com/BrowserBox/hyper-frame), [current embedding guide](https://docs.browserbox.io/embedding/)

<!-- Origin: web.md lines 95–95 -->

Browserless's self-hosted Live Debugger also explicitly provides an interactive visual screencast. This confirms availability of useful remote-browser interaction plumbing, not arbitrary slice extraction as a complete product. [Browserless Live Debugger](https://docs.browserless.io/enterprise/live-debugger)

<!-- Origin: web.md lines 97–97 -->

## 3. Why literal DOM transplantation is not a general solution

<!-- Origin: web.md lines 99–99 -->

`cloneNode()` omits addEventListener/property listeners and canvas painted contents. `importNode()` copies into a new document/custom-element registry; it is not a runtime-state serializer. Even perfect CSS capture cannot supply missing application behavior. [MDN cloneNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)

<!-- Origin: web.md lines 101–101 -->

Moving the original node instead of cloning may preserve its identity/listeners, but is still risky: document-level event delegation, DOM ancestry, styles, layout measurements, owner document, focus, observers, framework reconciliation, and custom-element lifecycle may change. The important distinction is between a DOM object and the application's effective component boundary. This is architectural analysis, not a claim that every node move fails.

<!-- Origin: web.md lines 103–103 -->

React itself warns that adding/removing/modifying nodes it manages can cause inconsistency or crashes and provides a concrete manual-removal example. Reactive apps can recreate the subtree or reapply styles after your mutation. An adapter can work with that behavior; a mutation observer that constantly “fights React” is not equivalent to correctness. [React DOM manipulation guidance](https://react.dev/learn/manipulating-the-dom-with-refs)

<!-- Origin: web.md lines 105–105 -->

React portals are different: app code renders JSX through a portal while preserving React ancestry/context and React event propagation. They require access to the executing component/tree/React integration point; an arbitrary external DOM selection does not become a React portal by being moved. [React createPortal](https://react.dev/reference/react-dom/createPortal)

<!-- Origin: web.md lines 107–107 -->

A mirror also needs non-HTML state. rrweb's serialization describes separately recording input state, rewriting URLs, inlining styles, assigning node IDs, and de-scripting replay. Its sandbox deliberately avoids rerunning original scripts. Replay therefore does not rerun the original application behavior. [rrweb serialization](https://github.com/rrweb-io/rrweb/blob/main/docs/serialization.md), [rrweb sandbox](https://github.com/rrweb-io/rrweb/blob/main/docs/sandbox.md)

<!-- Origin: web.md lines 109–109 -->

A current static-extraction counterexample is **DOM Capture**: it offers style-complete HTML and shadow-root support, but its own README explicitly says JavaScript behavior does not travel. It also discusses session-signed iframe sources. [DOM Capture](https://github.com/schappim/dom-capture)

<!-- Origin: web.md lines 111–111 -->

## 4. Research precedents: what was actually demonstrated

<!-- Origin: web.md lines 113–113 -->

### Fusion, UIST 2018

<!-- Origin: web.md lines 115–115 -->

Xiong Zhang and Philip Guo's Fusion is a Chrome-extension UI-mashup prototype: select from unmodified pages, iframe-transclude widgets, and connect them with JavaScript glue. Authors report seven case studies, each under 15 glue-code lines, including Python Tutor inside tutorials, code/documentation lookup, LaTeX preview, data-science tooling, and collaboration. This is author-reported demonstration evidence, not this memo's tested compatibility result.

<!-- Origin: web.md lines 117–117 -->

Crucial limitations: literal CSS selectors; iframe restrictions; finding actionable DOM/events in complex apps; no generic deeper backend integration. Their cross-domain setup used disabled same-origin checks for prototyping or a same-origin proxy alternative. The paper calls it medium-fidelity prototyping rather than production infrastructure. [Author-hosted full paper, especially pp. 6–9](https://pg.ucsd.edu/publications/Fusion-opportunistic-web-prototyping-UI-mashups_UIST-2018.pdf)

<!-- Origin: web.md lines 119–119 -->

### C3W / Clip, Connect, Clone

<!-- Origin: web.md lines 121–121 -->

Historical research implemented visual cells as portals to web pages, with recorded navigation and HTML/XPath-style addressing. Input/result cells were connected by spreadsheet-like formulas; cloning let users compare alternatives. The paper describes a specialized browser/PlexWare/IntelligentPad implementation, not a currently maintained universal modern-SPA library. Primary author paper available through ResearchGate: [Clip, connect, clone](https://www.researchgate.net/publication/220877248_Clip_connect_clone_Combining_application_elements_to_build_custom_interfaces_for_information_access), [short C3W paper](https://www.researchgate.net/publication/221023398_C3W_clipping_connecting_and_cloning_for_the_web)

<!-- Origin: web.md lines 123–123 -->

### WebMakeup

<!-- Origin: web.md lines 125–125 -->

A university thesis/project record describes a Chrome extension for selecting, moving, removing, and adding nodes from other pages; it explicitly identifies site updates/locator robustness as a difficulty. Modern arbitrary authenticated-SPA compatibility was not established. [University thesis record](https://www.ehu.eus/eu/web/doktoregoa/ingeniaritza-informatikoa-doktoregoa/defendatutako-tesiak)

<!-- Origin: web.md lines 127–127 -->

### Webstrates and MyWebstrates

<!-- Origin: web.md lines 129–129 -->

Webstrates synchronizes persistent DOM changes across clients and composes webstrates via transclusion. It is a powerful purpose-designed substrate; it does not prove arbitrary proprietary applications can be extracted with equivalent semantics. Its original paper says state not represented in the DOM is not shared. MyWebstrates (UIST 2024) advances a local-first version of the substrate. [Webstrates documentation](https://webstrates.github.io/), [original author paper](https://pure.au.dk/ws/files/91047333/webstrates.pdf), [MyWebstrates author paper](https://cs.au.dk/~clemens/files/MyWebstrates-UIST2024.pdf)

<!-- Origin: web.md lines 131–131 -->

### d.mix and Bricolage: adjacent, not equivalent

<!-- Origin: web.md lines 133–133 -->

Stanford's d.mix samples annotated sites into API calls: useful site-to-service translation but depends on a mapping to services/APIs. Bricolage transfers content into another page's style/layout using learned mappings. Neither is evidence of universally preserving hidden runtime behavior of arbitrary selected controls. [d.mix project](https://hci.stanford.edu/research/mashups/), [Bricolage author-hosted paper](https://hci.stanford.edu/publications/2011/Bricolage/Bricolage-CHI2011.pdf)

<!-- Origin: web.md lines 135–135 -->

For multiple source apps, there is no generic global undo or transaction. If action A succeeds and B fails, compositor-side rollback cannot assume A is reversible.

<!-- Origin: web.md lines 156–156 -->

## 6. Authentication/security boundaries that survive unlimited engineering effort

<!-- Origin: web.md lines 158–158 -->

**App-origin authentication is not just a URL string.** WebAuthn credentials are scoped to a relying-party ID and browser-checked origin constraints. Rewriting a website to a proxy origin does not automatically make its existing passkeys usable there. Related-origin requests are defined, but require the relying party's permitted-origin machinery; they are not unilateral arbitrary-proxy authorization. [WebAuthn Level 3](https://www.w3.org/TR/webauthn-3/)

<!-- Origin: web.md lines 160–160 -->

**Provider acceptance is separate from rendering ability.** Google's OAuth policy disallows authorization in developer-controlled embedded user agents and requires verifiable connection identity. An Electron browser can render HTML perfectly and still not be an acceptable login surface. A compliant external-browser login route may work when the application supports the redirect/token handoff; a third-party compositor cannot presume it can retrofit that into every SaaS service. [Google OAuth policies](https://developers.google.com/identity/protocols/oauth2/policies)

<!-- Origin: web.md lines 162–162 -->

**Sharing a browser session is not cloning app state.** Electron supports named persistent/in-memory session partitions. Sharing a partition provides shared session storage facilities where appropriate; it does not duplicate a page's current JS heap, unsaved input, selected route, or in-flight task. [Electron session API](https://www.electronjs.org/docs/latest/api/session)

<!-- Origin: web.md lines 164–164 -->

**Native trust does not imply renderer trust.** Electron's security guidance recommends sandboxing, context isolation, no Node integration for remote content, limited permissions/navigation/window creation, and validating IPC senders. These are essential because a composition app intentionally brings untrusted origins close to privileged desktop functionality. [Electron security guide](https://www.electronjs.org/docs/latest/tutorial/security)

<!-- Origin: web.md lines 166–166 -->

Not established by this research: universal authenticated-app compatibility; performance with dozens of slices; zero-copy rendering across every OS/GPU; complete accessibility/IME integration; browser/store approval for every extension feature; or automatic semantic adaptation across arbitrary vendor changes.

<!-- Origin: web.md lines 207–207 -->

## 9. Addendum: live-node movement, newer primitives, and generated adapters

<!-- Origin: web.md lines 211–211 -->

### Same-document moves and copying

<!-- Origin: web.md lines 213–213 -->

These operations differ:

<!-- Origin: web.md lines 215–215 -->

- **CSS-only rearrangement:** retains actual nodes and DOM ancestry. It can preserve most runtime and event structure but changes layout/geometry and may be overwritten by the app.
- **Moving the actual node within its original document:** preserves the node object rather than creating a copy. `appendChild(existingNode)` explicitly moves it; directly attached listeners are not the cloning problem described earlier. Old and new positions cannot both contain that same node. [appendChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)
- **Atomic same-document move:** `moveBefore()` is a newer browser primitive. Chrome introduced it in 133; it avoids the implicit remove/reinsert and preserves state such as iframe loading, focus, animations, fullscreen, popovers and modal dialogs. [Chrome introduction](https://developer.chrome.com/blog/movebefore-api)
- **Cross-document adoption:** `adoptNode()` transfers the same node, removes it from the old document, and changes `ownerDocument`. This is not cloning, but it changes important document dependencies. It does not move every other object the component's callbacks close over, or grant access across origin boundaries. [adoptNode](https://developer.mozilla.org/en-US/docs/Web/API/Document/adoptNode)
- **Cross-document clone/import:** new nodes; event/paint/state omissions discussed above. This is the weakest option for preserving behavior.

<!-- Origin: web.md lines 217–221 -->

`moveBefore()` only supports a move within one document and with compatible connected/disconnected status. It has a state-preserving custom-element callback mechanism, `connectedMoveCallback()`. Crucially, atomic DOM state preservation does not promise that a third-party framework tolerates the new tree shape, or that old ancestor-dependent CSS and JS still mean the same thing. [moveBefore reference](https://developer.mozilla.org/en-US/docs/Web/API/Element/moveBefore)

<!-- Origin: web.md lines 223–223 -->

**Concrete event-path failure:** React 17 moved most delegated listeners from `document` to the React root container; React separately installs listeners on portal containers. Moving a React-managed node outside its original root without creating a real React portal can therefore sever the expected event path even while the node and its direct listeners survive. This is a version/framework integration issue, not evidence that movement cannot work. [React 17 event-delegation design](https://legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html)

<!-- Origin: web.md lines 225–225 -->

Native event paths also reflect DOM/shadow-tree relationships; outside observers do not see closed-shadow internal nodes in `composedPath()`. [composedPath](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath)

<!-- Origin: web.md lines 227–227 -->

### HTML-in-Canvas: experimental API and security scope

<!-- Origin: web.md lines 229–229 -->

The current WICG HTML-in-Canvas explainer proposes browser-rendered DOM snapshots in 2D/WebGL/WebGPU plus geometry synchronization, accessibility, and hit-testing support. It is experimental/flagged; the API is evolving (older material uses `layoutsubtree`, current explainer uses `content="drawable"`/`drawable`). Security rules exclude information unavailable to author script, including cross-origin frame/content pixels and IME popups. It does not grant general cross-origin webpage capture/import permission. [WICG explainer](https://github.com/WICG/html-in-canvas), [Chrome origin-trial article](https://developer.chrome.com/blog/html-in-canvas-origin-trial)

<!-- Origin: web.md lines 231–231 -->

Framework introspection can reveal how UI state is represented. It cannot guarantee that a field named `id` means a customer rather than an invoice, that invoking a callback directly reproduces a trusted user action, that a backend operation is reversible, or that an invisible server precondition has been satisfied. Directly invoking an internal callback can also skip validation, batching, analytics, form defaults, focus, or app event sequencing.

<!-- Origin: web.md lines 244–244 -->
