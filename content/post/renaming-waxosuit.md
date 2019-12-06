---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Renaming Waxosuit to waSCC"
subtitle: "Wherein a prototype aims toward maturity"
summary: ""
authors: ["kevin"]
tags: ["waxosuit"]
categories: []
date: 2019-12-06T10:50:07-05:00
lastmod: 2019-12-06T10:50:07-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
Some of you who have found your way to this website might have seen references to something called **waxosuit** in your travels. When the original experiment started, I set out to create a "WebAssembly exosuit", a shell into which you could place your WebAssembly module that would then connect your code with cloud-native capabilities and satisfy your non-functional requirements.

The goal of that experiment (there is a whitepaper forthcoming) was to learn. I wanted to learn what was technically possible, what was difficult, what was easy, and most importantly, what people needed and wanted in the _"WebAssembly in the cloud"_ ecosystem.

Now that the original experiment is over and I am in the process of gathering up everything I've learned from it and using that to fuel the roadmap for **waSCC**, I thought it was worth at least posting to clear things up.

**Waxosuit** refers to the original experiment designed to build something rapidly, get it into people's hands rapidly, and learn from that usage and exposure. Waxosuit was not intended to be a final-stage product that people could take directly to production. It was a prototype.

That part of the experiment is over and we're now moving into the next phase. This is where **waSCC** becomes a suite of open source tools and libraries to facilitate the development of _WebAssembly actors_ that can run anywhere--in the cloud, at the edge, on your laptop, on embedded devices, as IoT hubs, and much more.