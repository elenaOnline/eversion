# Report: evidence and documented mechanisms

Editorial extraction, 2026-10-07. Source research was read-only; no listed software was installed, executed, or benchmarked. Documented APIs, author-reported demonstrations, source inspections, and untested compatibility are distinguished below. Source-level conclusions are limited to the investigated interfaces and versions. No implementation architecture is prescribed here.

**WinCuts, 2004.** Microsoft Research implemented live, interactive regions of existing windows as independent windows. Its remote-sharing extension was read-only, an important distinction from the local interactive version. It does not supply a universal domain model. [Paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2004/01/WinCuts.pdf)

<!-- Origin: report.md lines 153–153 -->

Neovim's remote UI protocol supports external clients, per-window grids and externalized interface elements. This is a cooperative engine interface; it does not demonstrate extraction from an unfamiliar application. [Neovim UI protocol](https://neovim.io/doc/user/api-ui-events/)

<!-- Origin: report.md lines 287–287 -->
