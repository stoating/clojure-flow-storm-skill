# FlowStorm Anti-Patterns

## Contents

- Instrumenting too much
- Recording too early
- Ignoring trace and heap limits
- Confusing compiler modes
- Mutable value surprises
- Macro and source mapping issues
- Remote setup mistakes

---

## Instrumenting Too Much

Do not start by instrumenting every dependency. Broad instrumentation creates noisy timelines, large memory retention, and slow UI updates.

Prefer:

```text
-Dclojure.storm.instrumentOnlyPrefixes=my-app,my-lib
-Dclojure.storm.instrumentSkipPrefixes=my-app.hot-path,my-lib.noisy
```

Avoid instrumenting:

- `clojure.core`
- FlowStorm namespaces
- logging internals
- very high-frequency UI/event loops unless using light instrumentation or limits

---

## Recording Too Early

Leaving recording on while booting a large app can fill recordings with startup noise. Start with:

```text
-Dflowstorm.startRecording=false
```

Then enable recording immediately before the action under investigation and disable it afterward.

---

## Ignoring Trace And Heap Limits

When debugging loops or high-frequency callbacks, configure fuses:

```text
-Dflowstorm.threadTraceLimit=1000
-Dflowstorm.throwOnLimit=true
-Dflowstorm.heapLimit=1000
```

Function call limits are useful for noisy functions:

```text
-Dflowstorm.threadFnCallLimits=org.my-app/fn1:2,org.my-app/fn2:4
```

Or adjust from the REPL:

```clojure
(require '[flow-storm.runtime.indexes.api :as ia])

(ia/add-fn-call-limit "org.my-app" "fn1" 10)
(ia/rm-fn-call-limit "org.my-app" "fn1")
(ia/get-fn-call-limits)
```

---

## Confusing Compiler Modes

ClojureStorm/ClojureScriptStorm instrumentation happens at compile time. If a namespace was already loaded before adding a prefix, reload it.

Vanilla FlowStorm instrumentation rewrites selected forms or namespaces. It needs explicit `#trace`, `#rtrace`, or API/browser instrumentation.

Do not mix advice from the wrong mode. For example, `#trace` is not how normal ClojureStorm code should be instrumented; ClojureStorm should use prefixes/browser controls and reloads.

---

## Mutable Value Surprises

FlowStorm records references. Immutable values are safe to inspect later, but mutable objects can show their later state instead of the state they had at the recorded step.

For mutable Java objects or app-specific references, snapshot them before recording or extend FlowStorm's snapshot protocol for the type.

---

## Macro And Source Mapping Issues

Macros that do not preserve metadata can make recorded forms appear under expanded code rather than the user's original source. If an expression inside a macro is missing, inspect macro expansion and source metadata before assuming FlowStorm failed.

Loading an entire file while recording may record loader/compiler activity. Clear recordings before running the scenario you actually care about.

---

## Remote Setup Mistakes

For remote CLJS or split runtime/UI setups:

- make sure the runtime has `flow-storm-inst`
- make sure the UI has `flow-storm-dbg`
- confirm nREPL port, websocket port, and host values
- for shadow-cljs, pass the correct `:build-id`
- reload the browser/runtime after starting the UI
- for React Native or devices, use the dev machine IP as `:debugger-host`

If GitHub Codespaces, Docker, WSL2, or SSH is involved, check port forwarding before changing FlowStorm configuration.
