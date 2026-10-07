# Critique: evidence and documented mechanisms

Editorial extraction, 2026-10-07. Source research was read-only; no listed software was installed, executed, or benchmarked. Documented APIs, author-reported demonstrations, source inspections, and untested compatibility are distinguished below. Source-level conclusions are limited to the investigated interfaces and versions. No implementation architecture is prescribed here.

Frida documents function interception, calling native functions, Objective-C runtime access and implementation replacement. This is a concrete mechanism for exposing application behavior beyond pixels or accessibility. It does not automatically identify safe domain semantics, and only applies where execution/instrumentation is actually permitted. [Frida JavaScript API](https://frida.re/docs/javascript-api/)

<!-- Origin: critique.md lines 37–37 -->

Chrome's debugger extension API can inspect JavaScript, mutate DOM/CSS and instrument network interactions. Its exposed protocol domains include Runtime, Debugger, Input, DOM and Accessibility. It also documents enterprise restrictions that can refuse attachment. This is a much richer authorized channel than DOM scraping alone. [Chrome debugger API](https://developer.chrome.com/docs/extensions/reference/api/debugger)

<!-- Origin: critique.md lines 39–39 -->

The precise limit is: two states indistinguishable through **every permitted channel available to the implementation** cannot be distinguished reliably by that implementation. 

<!-- Origin: critique.md lines 43–43 -->

Server-only objects, inaccessible account data and remote commit state may remain unavailable. A client can have a pending optimistic update without knowing whether the server durably committed it. Those are genuine authority/observation boundaries.

<!-- Origin: critique.md lines 45–45 -->

Apple explicitly documents AXUIElementPostKeyboardEvent as targeting a specified application rather than always the active application. It accepts an application or system-wide accessibility object; it does not accept an arbitrary control as the target. CGEvent also exposes postToPid. Neither API documents a complete independent input session for each crop. [AX targeted keyboard input](https://developer.apple.com/documentation/applicationservices/1462057-axuielementpostkeyboardevent), [CGEvent](https://developer.apple.com/documentation/coregraphics/cgevent)

<!-- Origin: critique.md lines 51–51 -->

AppKit routes ordinary keyboard events through key equivalents, interface navigation, the key window and first responder. Mouse handling has its own hit-testing and gesture ownership. Target/action dispatch can take another responder path. Therefore “the event reached the PID” is a weaker success condition than “the intended component behaved exactly as if hosted normally.” The archived architecture guide explains these dependencies; modern behavior must still be measured. [Apple Event Architecture](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/EventOverview/EventArchitecture/EventArchitecture.html)

<!-- Origin: critique.md lines 53–53 -->

Windows' AttachThreadInput provides a concrete example of input state being shared across threads, including current focus. It has constraints and resets key state when called. [Microsoft AttachThreadInput](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-attachthreadinput)

<!-- Origin: critique.md lines 67–67 -->

Qt's window-container documentation similarly describes embedding with platform-dependent focus activation, focus-return responsibilities, stacking and rendering limitations. Even cooperative framework embedding has an input contract to implement. [Qt QWidget/window containers](https://doc.qt.io/qt-6.8/qwidget.html)

<!-- Origin: critique.md lines 69–69 -->

The current official guide, updated September 30, 2026, documents comments, writing in suggestion mode, and accepting/rejecting/deleting suggestion threads. It also warns that comment/suggestion persistence may fail while document-model changes succeed. [Google Docs comments and suggestions](https://developers.google.com/workspace/docs/api/how-tos/suggestions)

<!-- Origin: critique.md lines 142–142 -->

The current manual explicitly distinguishes tempo/beat/phase synchronization from real-time Link Audio streaming among compatible peers. “Link never carries audio” is no longer correct. This does not imply transport of arbitrary Live project state or plugin internals. [Ableton Link and Link Audio](https://www.ableton.com/en/manual/synchronizing-with-link-tempo-follower-and-midi/)

<!-- Origin: critique.md lines 146–146 -->

Apple's current sandbox documentation explicitly lists assistive Accessibility API use as incompatible. A June 2025 DTS answer reiterates sandbox incompatibility and the Mac App Store sandbox requirement. This differs from exposing an application’s own accessible UI. [Apple sandbox documentation](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox), [Apple DTS answer](https://developer.apple.com/forums/thread/789663)

<!-- Origin: critique.md lines 150–150 -->

Chrome's active-session MCP auto-connect flow is documented for Chrome 144+, after the user enables remote debugging. Each connection request in that flow gets a permission dialog; an active session shows an automation banner. Other connection modes still exist. The described consent behavior applies to the documented auto-connect route; it does not establish identical UI across every debugger transport/profile. [Chrome active-session debugging](https://developer.chrome.com/blog/chrome-devtools-mcp-debug-your-browser-session)

<!-- Origin: critique.md lines 154–154 -->
