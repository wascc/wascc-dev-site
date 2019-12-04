+++
# A Skills section created with the Featurette widget.
widget = "blank"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 20  # Order that this section will appear.

title = "waSCC"
subtitle = ""

# Showcase personal skills or business features.
# 
# Add/remove as many `[[feature]]` blocks below as you like.
# 
# For available icons, see: https://sourcethemes.com/academic/docs/widgets/#icons

#[[feature]]
#  icon = "r-project"
#  icon_pack = "fab"
#  name = "R"
#  description = "90%"
+++

**waSCC**, _WebAssembly Secure Capabilities Connector_, is a WebAssembly host runtime that dynamically and securely binds **WebAssembly** modules to capability providers. These provider plug-ins provide abstractions around and connectivity to cloud-native services—message brokers, databases, HTTP—or they can expose special-purpose capabilities like IoT, embedded hardware, and more.

WebAssembly is taking the web by storm, but we believe that WebAssembly truly shines in the cloud. waSCC is designed around the following core tenets:

* Productivity - Developer and Operations
* Enterprise-grade Security
* Cost Savings
* Portability
* Performance

waSCC is built upon a set of standards and conventions that attempt to fill the void for cloud-native Wasm module host environments. Using this host runtime, you can host WebAssembly modules as standalone processes on a laptop, as on-demand functions in a serverless environment, as always-available microservices in the cloud or at the edge, or treat `.wasm` files like firmware on Raspberry Pis and embedded devices.
