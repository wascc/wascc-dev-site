---
title: "Assembly Mechs: Beyond WasmDome"
summary: An online, realtime multiplayer game written in WebAssembly
tags:
- space
- gaming
date: "2019-10-27T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

links:
- icon: twitter
  icon_pack: fab
  name: Follow waSCC
  url: https://twitter.com/wascc_runtime
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
#slides: example
---

**Assembly Mechs: Beyond WasmDome** is a game experiment we launched in order to learn more about waSCC, gather feedback from players and developers, and validate our assumptions about our WebAssembly actor runtime.

{{< figure library="true" src="wasmdome_match.gif" title="WasmDome Match" numbered="false" lightbox="true" >}}

The back-end engine of this project was created using waSCC actors and the competitive robots created by player-developers were also waSCC actors. Another enabling technology that made this game possible was [lattice](/docs/lattice/overview).

For more information on the process we went through in designing the game, refactoring its architecture, implementing it, and building it out online, check out the various relevant documentation sections.

## Tutorials

The following are two video tutorials that we created for building mechs, running offline matches, and finally competing online in a lattice-based arena.

[![Assembly Mechs: Beyond Wasmdome Offline Tutorial](https://img.youtube.com/vi/xjy61n7frHo/0.jpg)](http://www.youtube.com/watch?v=xjy61n7frHo "Assembly Mechs: Beyond Wasmdome Offline Tutorial")

[![Assembly Mechs: Beyond Wasmdome Online Tutorial](https://img.youtube.com/vi/PBQ1tyeXrCA/0.jpg)](http://www.youtube.com/watch?v=PBQ1tyeXrCA "Assembly Mechs: Beyond Wasmdome Online Tutorial")