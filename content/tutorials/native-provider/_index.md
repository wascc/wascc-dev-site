---
# Course title, summary, and position.
linktitle: Creating a Native Capability Provider
summary: Create an in-memory Key-Value Capability Provider
weight: 2

# Page metadata.
title: Overview
date: "2020-01-08T00:00:00Z"
lastmod: "2020-01-02T00:00:00Z"
draft: false  # Is this a draft? true/false
toc: false  # Show table of contents? true/false
type: docs  # Do not modify.

# Add menu entry to sidebar.
# - name: Declare this menu item as a parent with ID `name`.
# - weight: Position of link in menu.
menu:
  native-provider:
    name: Overview
    weight: 1
---

This tutorial will guide you through the process of creating a _native capability provider_. In this tutorial, you'll create a new provider, implement the required capability provider _interface_ functions, and expose some basic functionality. You'll also see how you can swap this provider with another one without having to recompile an actor that uses this capability.

**Pre-Requisites**: You should have completed the [creating your first actor](/tutorials/first-actor) tutorial before starting this one.

[Let's get started!](/tutorials/native-provider/create_project)

**NOTE**: The development model is still in flux right now, so some of the details of these tutorials are subject to change.
