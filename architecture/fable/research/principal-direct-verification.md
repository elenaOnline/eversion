# Principal direct verification notes

Author: the Fable architect (principal), not a subagent. Date: 2026-10-07. Method: WebFetch of Apple developer documentation. Apple's HTML pages render client-side and returned only titles; the same content is served as JSON under the documentation data path, which is what was read. No software was installed or executed. Tags: [D] documented behavior, [I] inference.

These are the small number of primary-source checks the principal made directly because they are load-bearing for the architecture and were not assigned to a research subagent.

## 1. ScreenCaptureKit stream configuration properties

Source: SCStreamConfiguration reference, https://developer.apple.com/documentation/screencapturekit/scstreamconfiguration

- [D] Class available macOS 12.3+. Properties include `width`, `height`, `scalesToFit`, `sourceRect`, `destinationRect`, `preservesAspectRatio`, `pixelFormat`, `colorMatrix`, `colorSpaceName`, `backgroundColor`, `showsCursor`, `shouldBeOpaque`, `capturesShadowsOnly`, `ignoreShadowsDisplay`, `ignoreShadowsSingleWindow`, `ignoreGlobalClipDisplay`, `ignoreGlobalClipSingleWindow`, `queueDepth`, `minimumFrameInterval`, `captureResolution`, `capturesAudio`, `sampleRate`, `channelCount`, `excludesCurrentProcessAudio`, `streamName`, `presenterOverlayPrivacyAlertSetting`, plus `captureDynamicRange`, `captureMicrophone`, `includeChildWindows`, `microphoneCaptureDeviceID`, `showMouseClicks`, and a preset initializer `init(preset:)`.
- [D] `ignoreGlobalClipSingleWindow` (macOS 14.0+): "A Boolean value that indicates if the stream ignores content clipped past the edge of a display, when streaming in window style." Discussion: "The display that originates the stream determines clipping bounds. When this value is true, the stream contains content moved past the clipping bounds. The default value is false." Source: https://developer.apple.com/documentation/screencapturekit/scstreamconfiguration/ignoreglobalclipsinglewindow
- [D] `includeChildWindows` exists with availability macOS 14.2+ and Mac Catalyst 18.2+. Apple's reference page for it carries no description text. Source: https://developer.apple.com/documentation/screencapturekit/scstreamconfiguration/includechildwindows
- [I] The name suggests that a window-style stream can include child windows (AppKit attaches popovers, sheets, and some menus as child windows of their parent). Whether context menus, input-method candidate panels, and tooltips count as child windows is not documented and must be tested. This is listed as an experiment in the architecture.
- [I] `ignoreGlobalClipSingleWindow` gives a public-API route to keep a source window mostly outside the visible display area while still capturing its full content. Whether AppKit or the target app will accept a window frame that lies mostly off-screen is not established by documentation (see section 4) and must be tested.

## 2. ScreenCaptureKit content filters and shareable content

Sources: https://developer.apple.com/documentation/screencapturekit/sccontentfilter and https://developer.apple.com/documentation/screencapturekit/scshareablecontent

- [D] Filter initializers: `init(desktopIndependentWindow:)` "captures only the specified window"; `init(display:including:exceptingWindows:)` "captures a display, including only windows of the specified apps"; `init(display:excludingApplications:exceptingWindows:)`; `init(display:including:)` for specific windows from a display; `init(display:excludingWindows:)`. Properties include `includeMenuBar`, `includedDisplays`, `includedApplications`, `includedWindows`, `contentRect`, `pointPixelScale`, `style`, `isCameraEnabled`, `isMicrophoneEnabled`. Several newer members have no description text on the reference page.
- [D] `SCShareableContent.getExcludingDesktopWindows(_:onScreenWindowsOnly:completionHandler:)` (macOS 12.3+): `onScreenWindowsOnly` is "A Boolean value that indicates whether to include only onscreen windows in the set of shareable content." Apple does not define "onscreen" on that page and does not say whether `false` yields minimized windows or windows on other Spaces.
- [I] Display-style capture with an application inclusion list is the mechanism by which a region of a staging display can be captured together with transient windows that belong to other processes (for example input-method candidate panels), provided those processes are included in the filter. The findings packet already establishes that `sourceRect` is honored for display capture and ignored for single-window capture.

## 3. Vision and VisionKit on-device text and interaction overlays

Sources: https://developer.apple.com/documentation/vision/recognizing-text-in-images and https://developer.apple.com/documentation/visionkit/imageanalyzer

- [D] `VNRecognizeTextRequest` via `VNImageRequestHandler` recognizes text in a `CGImage`. Two paths: fast ("similar to traditional optical character recognition") and accurate ("uses a neural network to find text in terms of strings and lines"); accurate is the default. Results are `VNRecognizedTextObservation` objects; `topCandidates(_:)` returns `VNRecognizedText` with a `string` and a confidence score; `boundingBox(for:)` returns a normalized rectangle for a string range. "In all cases, all of Vision's processing happens on the user's device to enhance performance and user privacy." Language support depends on path and revision; `recognitionLanguages` sets priority order; Chinese has documented restrictions on language correction.
- [D] VisionKit `ImageAnalyzer` is available on macOS 13.0+. It "finds items in images that people can interact with, such as subjects, text, and QR codes." It accepts `NSImage`, `CGImage`, `CIImage`, `CVPixelBuffer`, and `URL` inputs. `isSupported` reports device support; `supportedTextRecognitionLanguages` lists languages. On macOS the documented presentation is `ImageAnalysisOverlayView`, "a view that enables people to interact with recognized text, barcodes, and other objects in an image," placed above the view containing the image, with `analysis` and `preferredInteractionTypes` properties and a delegate.
- [I] Because ScreenCaptureKit frames arrive as `CVPixelBuffer`s, a pixel-only cell can be given selectable, copyable text through documented on-device APIs without any access to the source app's text model. This is a presentation affordance, not semantic access; the recognized text is not the app's authoritative string.

## 4. AppKit window frame constraint

Source: https://developer.apple.com/documentation/appkit/nswindow/constrainframerect(_:to:)

- [D] `constrainFrameRect(_:to:)` "Modifies and returns a frame rectangle so that its top edge lies on a specific screen." It is "invoked automatically ... whenever a titled NSWindow object is placed onscreen and whenever its size is changed." Width and horizontal location are "unaffected." Subclasses can override it.
- [I] This constrains the top edge of a titled window to a screen when AppKit places it. The documentation does not say whether frames set through the Accessibility position attribute, which the target app's own AppKit processes, are subject to the same constraint. Pushing a source window mostly off-screen horizontally appears compatible with the documented rule (horizontal location unaffected); pushing it mostly off-screen vertically does not. This must be tested per app.

## 5. Foundation Models framework

Source: https://developer.apple.com/documentation/foundationmodels

- [D] Available macOS 26.0+. "The Foundation Models framework provides access to any large language model, like the on-device and Private Cloud Compute models designed for Apple Intelligence." Key types: `SystemLanguageModel` ("An on-device Apple Foundation Model capable of text generation tasks"), `LanguageModelSession`, the `@Generable` macro for guided generation of Swift data structures, `Tool` ("A tool that a model can call to gather information at runtime or perform side effects"), `PrivateCloudComputeLanguageModel`, and a `LanguageModel` protocol with `LanguageModelExecutor` types for "Running a Core AI model in a Foundation Models session." Dynamic profiles are described as letting developers "build many useful abstractions, such as agents or skills." Multimodal image prompting is listed under "Prompt attachments." Requires a device that supports Apple Intelligence.
- [D] No numeric context window, rate limit, or model size is stated on the overview page; a separate article "Managing the context window" exists and was not read.
- [I] For the architecture this establishes a documented, on-device, structured-output model runtime with tool calling on current macOS, and a provider protocol through which other models can be slotted in. It does not establish capability or reliability for the tasks the architecture assigns to agents; those are experiments.
