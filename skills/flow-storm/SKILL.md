---
name: flow-storm
description: Use when working with FlowStorm - an omniscient time-travel debugger for Clojure and ClojureScript. Activate when the user asks about ClojureStorm, ClojureScriptStorm, flow-storm-dbg, flow-storm-inst, flow-storm.api/local-connect, :dbg, :rec, :stop, #trace, #rtrace, #tap, instrumenting vars or namespaces, FlowStorm browser instrumentation, nREPL middleware, Outputs, Data Windows, datafy/nav, thread breakpoints, remote debugging, trace limits, heap limits, shadow-cljs FlowStorm setup, or debugging recorded execution flows.
version: 1.0.0
---

# FlowStorm

FlowStorm is an omniscient tracing debugger for Clojure and ClojureScript. It instruments code, records expression values and function calls as execution flows, then lets the user inspect the recorded timelines with stepping, searching, call trees, data windows, taps, and programmable APIs.

Artifacts:

- `com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}` - full debugger UI plus runtime
- `com.github.flow-storm/flow-storm-inst {:mvn/version "4.5.9"}` - slim runtime for remote/debuggee processes
- `com.github.flow-storm/clojure {:mvn/version "1.12.4"}` - ClojureStorm compiler
- `com.github.flow-storm/clojurescript {:mvn/version "1.12.134-3"}` - ClojureScriptStorm compiler

Prerequisites: JDK 17+ for the UI and Clojure 1.11+ for normal artifact usage.

## Quick Decision: What do you need?

| Task | Go to |
|------|-------|
| Set up ClojureStorm, vanilla FlowStorm, deps.edn, Leiningen, or start the UI | [setup.md](setup.md) |
| Set up ClojureScriptStorm, shadow-cljs, cljs.main, or vanilla CLJS debugging | [clojurescript.md](clojurescript.md) |
| Choose instrumentation strategy, use `#trace` / `#rtrace`, instrument vars/namespaces, or tune prefixes | [instrumentation.md](instrumentation.md) |
| Use flows, stepping, Outputs, taps, data windows, datafy/nav, bookmarks, or thread breakpoints | [debugging-workflow.md](debugging-workflow.md) |
| Debug over nREPL, SSH tunnels, split runtime/UI processes, Docker, or remote ClojureScript | [remote-and-runtime.md](remote-and-runtime.md) |
| Avoid too many traces, memory growth, mutable value surprises, macro issues, or unsupported instrumentation | [anti-patterns.md](anti-patterns.md) |

## Core Mental Model

```text
instrumented code -> execution -> recorded timelines -> FlowStorm UI / API
```

- Prefer ClojureStorm or ClojureScriptStorm for development when possible; they swap the compiler only in dev and automatically instrument code.
- Use vanilla FlowStorm when compiler swapping is not available or when you want manual `#trace` / `#rtrace` instrumentation.
- Keep recording off until just before the action you care about when the program is noisy.
- Limit instrumentation prefixes early. Do not instrument `clojure.core`, FlowStorm internals, or broad dependency trees without a reason.
- Clear recordings and use trace/heap limits when debugging loops, UI events, streams, or high-frequency functions.
