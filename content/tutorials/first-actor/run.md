---
title: Running the Host
linktitle: Run the Host
toc: true
type: docs
date: "2019-12-09T00:00:00Z"
draft: false
menu:
  first-actor:
#    parent: Example Topic
    weight: 4

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

To run this sample, first make sure you don't have anything currently using the local network port `8081` and then do `cargo run` with the `RUST_LOG` environment variable set to `INFO`. If this environment variable is unset or set to a less verbose level, then you won't see much information, if any at all, on your console.

## Running the Program

```shell
$ RUST_LOG=info cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.10s
     Running `target/debug/hellorunner`
[2019-12-10T15:35:04Z INFO  wascc_host::authz] Discovered capability attestations: wascc:http_server
[2019-12-10T15:35:04Z INFO  wascc_host::host] Adding actor MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR to host
[2019-12-10T15:35:04Z INFO  wascc_host::host] Loading actor module...
[2019-12-10T15:35:13Z INFO  wascc_host::host] Actor MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR ready for communications, capability: false
[2019-12-10T15:35:13Z INFO  wascc_host::capability] Loaded capability: wascc:http_server, native provider: waSCC Default HTTP Server (Actix Web)
[2019-12-10T15:35:13Z INFO  wascc_httpsrv] Dispatcher configured.
[2019-12-10T15:35:13Z INFO  wascc_host::host] Native capability provider 'wascc:http_server' ready
[2019-12-10T15:35:13Z INFO  wascc_host::host] Attempting to configure actor MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR for capability wascc:http_server
[2019-12-10T15:35:13Z INFO  wascc_host::host] Capability wascc:http_server received invocation for target wascc:http_server
[2019-12-10T15:35:13Z INFO  wascc_httpsrv] Handling operation `Configure` from `system`
[2019-12-10T15:35:13Z INFO  wascc_httpsrv] Received HTTP Server configuration for MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR
[2019-12-10T15:35:13Z INFO  actix_server::builder] Starting 8 workers
[2019-12-10T15:35:13Z INFO  actix_server::builder] Starting server on 0.0.0.0:8081
```
Depending on the version of `wascc-host` you're using, your output may differ a little bit, but the basic idea should remain the same. We can see that we're adding our `MCYQS...` actor to the host runtime, and we're adding a native capability provider for the `wascc:http_server` capability. 

Next, you can see the call to `Configure` being managed by the host runtime, setting up the instance of an HTTP server for our actor. Immediately after that, you can see the HTTP server (in our case, _Actix Web_) bring up a new instance.

To exercise your brand new microservice that has nearly _zero boilerplate_, issue the following `curl` command:

```shell
$ curl localhost:8081 | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    26  100    26    0     0  26000      0 --:--:-- --:--:-- --:--:-- 26000
{
  "data": 21,
  "hello": "world"
}
```

The `jq` command just gives us some nice printing (as well as powerful querying) JSON capabilities. 

You should see output that looks similar to the following in your service's terminal window:
```shell
[2019-12-10T15:38:43Z INFO  wascc_host::host] Capability wascc:http_server received invocation for target MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR
[2019-12-10T15:38:43Z INFO  wapc] Wasm Guest: Performing guest call, operation - HandleRequest
[2019-12-10T15:38:43Z INFO  actix_web::middleware::logger] 127.0.0.1:33558 "GET / HTTP/1.1" 200 26 "-" "curl/7.65.3" 0.001570
```

### Troubleshooting

Errors like this mean that one or more of your actor or module file paths are incorrect:

```
Error: Error(IO(Os { code: 2, kind: NotFound, message: "No such file or directory" }))
```

An error like this means that you forgot to sign your actor's module, or that you are pointing to `hello.wasm` instead of `hello_signed.wasm`:

```
Error: Error(Authorization("No embedded JWT in actor module"))
```

This error indicates that the module key used in `host::configure()` is not correct:

```
Error: Error(Authorization("Actor MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR is not authorized to use capability wascc:http_server, configuration rejected"))
```

## Congratulations
You have built your first service using **waSCC**. Now it's time to explore the library some more and, more importantly, explore the possibilities of building secure services in the cloud, at the edge, or anywhere else with WebAssembly.