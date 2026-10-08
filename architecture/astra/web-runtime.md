# Owned browser runtime: supporting architecture memo

Recommend an owned Chromium execution service with a native macOS presentation and interaction layer. CEF can validate the first rendering path, but Eversion’s distinctive capability should be an engine-level **projection API** connecting source identity, paint output, hit testing, accessibility, and lifecycle. Keep attachment to existing browser sessions as a permanent second execution mode.

The following are architectural proposals, not demonstrated implementations.

## 1. Retain the execution closure; extract its presentation

A selected region remains inside its original document, JavaScript realm, framework tree, origin, frame relationships, storage context, workers, and network session. Initially retain the entire page and its dependencies; do not claim to infer the smallest sufficient closure. The host gets a durable projection handle rather than a transplanted DOM node.

Chromium already separates painted content, compositor frames, and display aggregation. Its compositor uses transform, clip, and effect properties; a `SurfaceLayer` references another producer’s frames. These are strong insertion points, but do not already provide arbitrary independently interactive DOM projections. [Chromium compositor architecture](https://chromium.googlesource.com/chromium/src/+/main/docs/how_cc_works.md)

An Eversion fork could associate a `ProjectionRoot` with selected layout objects, emit selected paint contributions into a separate output, retain original ancestry for layout and event propagation, and publish the corresponding source-coordinate hit-test map. This would require engineering beyond copying an existing composited layer: DOM boundaries, painting boundaries, clipping, blending, and compositing boundaries differ.

Treat “semantic closure” as a retention and dependency contract, not proof of portability. A chart may depend on its whole dashboard’s state store; preserving that store is acceptable even when only the chart is displayed.

## 2. Make every projection a versioned interaction contract

A projection should publish:

- Source execution identity: context, document epoch, frame, origin, principal, and source object anchor.
- Presentation identity: frame sequence, surface generation, device scale, transform, clip, and projected bounds.
- Interaction identity: hit-test data, focus owner, pointer capture, text-input state, and available semantic commands.
- Dependencies: owning document, related popups, dialogs, menus, and overlays that must appear during interaction.
- State: live, stale, detached, awaiting authentication, invalidated, or recoverable.

Input references the presented frame generation. If source geometry changes between presentation and dispatch, the engine revalidates the intended target or declines the event. A blind coordinate click against a newer layout must never count as the same operation.

Source coordinates remain canonical inside the page. Eversion translates native events into those coordinates and translates caret, IME candidate-window, drag-image, and accessibility geometry back out. Arbitrary shape clipping is feasible visually, but keyboard navigation still needs a defined path through projected and excluded controls.

Popup handling is part of composition. A menu spawned outside the selected DOM subtree must become a related transient surface, or temporarily expand into the complete source context. React explicitly permits portals whose DOM location differs from their logical ancestry; events follow the React tree. Consequently, DOM containment alone cannot identify a component’s interaction envelope. [React portal semantics](https://react.dev/reference/react-dom/createPortal)

## 3. Separate four meanings of “another view”

| Requested behavior | Execution model |
|---|---|
| Two locations show the same live control | Two projections of one source; selection, modal state, and underlying changes are shared |
| Two regions of one page are visible | Distinct projection anchors; shared document interaction state |
| Two independent searches or scroll positions | Separate page instances, optionally sharing an account storage context |
| Two alternative edits to one object | Explicit application-level branches or Eversion-owned proposal state |

CEF supports creating request contexts with shared storage. This provides an implementation primitive for account-sharing instances; it does not provide isolated server state or independent business objects. [CEF request-context interface](https://github.com/chromiumembedded/cef/blob/master/include/cef_request_context.h)

Do not make simultaneous independent viewports over one arbitrary document a baseline promise. Page code can observe viewport dimensions, element geometry, focus, selection, and scroll. Giving one document two incompatible answers changes program semantics. A browser fork might provide multiple presentation transforms without changing layout; independent responsive layouts require new execution instances or a verified framework-specific adapter.

Sharing storage also allows deliberate application coordination, including background behavior. Distinguish account sharing, document sharing, and view sharing in the runtime contract.

## 4. Use framework instrumentation to deepen capabilities, not to establish basic correctness

Engine instrumentation can reliably identify document epochs, navigation, DOM replacement, geometry, user interaction, and some observable effects. Framework adapters can add component ownership, model identity, action dispatch, and subscription semantics.

A React adapter might find the state/action boundary behind a chart and expose `setFilter`, `selectedRecord`, and `openDetails`; a rich-text adapter might expose logical ranges and edit transactions. Agents should discover candidate mappings and test them through controlled differential observations, then emit an adapter with explicit assumptions and invalidation rules.

Calling a discovered closure is weaker evidence than knowing its domain contract. Generated adapters must state whether success means “handler ran,” “optimistic model changed,” or “remote commit confirmed.” Framework upgrades invalidate evidence, not merely selectors.

## 5. Own GPU lifetime explicitly

Stock CEF’s accelerated callback exposes macOS IOSurface-backed rendering, but its current contract says the pooled resource cannot be cached or accessed after the callback and should be copied into a client-owned texture. Therefore the conservative claim is **GPU-to-GPU copying without CPU readback**, not a stock CEF zero-copy architecture. [CEF rendering contract](https://github.com/chromiumembedded/cef/blob/master/include/cef_render_handler.h)

An owned engine path could export leased resources with acquire synchronization and release acknowledgment after the host’s Metal work completes. Chromium’s transferable-resource source explicitly distinguishes sync-token, GPU-completion, and release-fence lifetime modes. These are concrete evidence that resource return and GPU completion must be coordinated. [Chromium transferable resources](https://raw.githubusercontent.com/chromium/chromium/main/components/viz/common/resources/transferable_resource.h)

Bound the outstanding frame queue. Slow consumers should drop intermediate frames, preserve the newest valid frame, and signal staleness. Include color space, alpha interpretation, damage, orientation, and scale in the contract. GPU reset or renderer death revokes every affected lease and projection generation. Chromium’s surface identity already separates a producer from sequential surface generations associated with size and scale. [Chromium surface identity](https://raw.githubusercontent.com/chromium/chromium/main/components/viz/common/surfaces/surface_id.h)

## 6. Preserve origin isolation while composing across origins

Web applications remain separate top-level browser contexts in their original origins. Composition happens in privileged native host space; it does not merge source pages into a shared JavaScript principal. Cross-app transfers go through explicit host capabilities.

Chromium’s OOPIF architecture tracks frames and origins in the browser process and combines rendering, input, and accessibility across renderer processes. This supports the architectural direction; it does not imply every DOM subtree is already a separately routable frame. [Chromium OOPIF design](https://www.chromium.org/developers/design-documents/oop-iframes/)

A projection may show source A beside source B without granting A access to B. The composition definition records separately what a human can see, what an agent may read, and which actions an agent may invoke. Agent-generated UI must not receive arbitrary browser-debugger authority merely because it displays a projection.

## 7. Authentication makes two execution modes necessary

Owned browser contexts offer maximal rendering control but establish their own sessions. Existing-session attachment preserves the actual tab, unsaved app state, and established authentication but offers less presentation control.

Chrome documents an active-session debugging route with prior enablement, a permission dialog for each connection request, and an active-control banner. Separately, Chrome 136 stopped honoring debugging startup switches for the default data directory. Neither fact supports silently importing the ordinary browser profile. [Active-session attachment](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session), [debugging-profile restriction](https://developer.chrome.com/blog/remote-debugging-port)

OAuth is especially important. RFC 8252 requires external user-agents for native-app authorization and PKCE for public native clients. That flow returns authorization for the registered client; it does **not** establish arbitrary third-party website cookies in Eversion’s owned context. [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252)

Google’s policy forbids directing OAuth authorization into developer-controlled embedded user-agents, explicitly including environments that permit arbitrary script insertion or cookie access. Calling the host a browser or opening an external sign-in window does not, by itself, resolve this. [Google OAuth policy](https://developers.google.com/identity/protocols/oauth2/policies)

Test authentication as a product-critical capability: ordinary login, federated login, passkeys, popup callbacks, enterprise policies, account switching, session expiry, and authentication after renderer recovery. Where owned execution cannot establish an accepted session, retain the authorized original browser session. Do not design around credential copying.

## 8. A changing workflow should rebind capabilities rather than rearrange screenshots

Consider a procurement investigation. Selecting a supplier in an original SaaS table instantiates linked projections of its contract, shipment history, and incident discussion. An agent identifies the supplier entity and proposes a mapping between otherwise unrelated identifiers. The user corrects one match; Eversion records the correction as an explicit relationship.

A delayed shipment adds an exception branch and a comparison view. Those comparisons are separate browser instances or typed host views, while the original editor keeps its unsaved draft. A new incident changes the workflow’s state, highlights affected evidence, and invalidates a previously generated summary. The agent prepares a proposed response, associates each claim with source versions, and presents the original application’s send capability. The final write executes against the current destination and records its observed outcome.

Projection, semantic mapping, branching, proposal state, and external commitment are separate mechanisms. That separation enables rich evolution without pretending every application shares one data model.

## 9. Experiments that could overturn this recommendation

- **Projection correctness:** Use controlled apps with nested scrolling, sticky positioning, transform effects, shadow DOM, canvas, cross-origin frames, portal menus, drag-and-drop, IME, and screen readers. Compare event traces and resulting state against normal rendering. Fail the generic projection claim if correctness requires changing ordinary app semantics.
- **Interaction ownership:** Two projections share a document while a human edits and an agent acts elsewhere. Force modal creation and focus theft. The experiment succeeds only if contention is visible and no action reaches an unintended target.
- **Lifetime torture:** Resize continuously, vary display scale, delay GPU completion, kill renderers, and reset the GPU process. Any stale resource access, frame misassociation, or unbounded retained memory falsifies the proposed resource protocol.
- **Identity repair:** Replace DOM subtrees, reorder identical-looking records, navigate away and back, and change accounts. Incorrect silent rebinding is a failure; deliberate suspension is acceptable.
- **Authentication acceptance:** Test representative real authentication systems before assuming the owned browser can be the dominant execution substrate.
- **Framework generalization:** Derive adapters for several unrelated apps using the same framework. Measure which capabilities survive app and framework updates. If most require app-specific reverse engineering, keep the engine generic but price semantic integration honestly.
- **Workflow continuity:** Sustain a workflow through source reload, auth expiry, optimistic-write failure, and concurrent human edits. A polished initial composition does not pass.

CEF is the shortest path to testing native GPU composition; an owned Chromium fork offers the strongest eventual control over projection semantics. WebKit remains an alternative engine, but no evidence reviewed here establishes a ready-made arbitrary projection API there. The decisive uncertainties are application compatibility, authentication acceptance, and whether engine-owned interaction can preserve source behavior under dynamic projection—not whether several rendered rectangles can be displayed together.

Research performed on 2026-10-07. No implementation, application execution, system changes, or Git writes were performed. This is supporting research; it is not the principal’s frozen initial architecture.
