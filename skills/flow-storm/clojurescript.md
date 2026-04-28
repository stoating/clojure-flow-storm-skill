# FlowStorm ClojureScript

## Contents

- Choosing ClojureScriptStorm vs vanilla
- ClojureScriptStorm with shadow-cljs
- ClojureScriptStorm with cljs.main
- Vanilla shadow-cljs
- Multiple builds
- Common CLJS limitations

---

## Choosing A CLJS Mode

Use ClojureScriptStorm when shadow-cljs resolves dependencies through `deps.edn`; it swaps the ClojureScript compiler and automatically instruments code.

Use vanilla FlowStorm when:

- shadow-cljs dependencies are declared directly in `shadow-cljs.edn`
- compiler swapping is not possible
- you only need manual tracing with `#trace` / `#rtrace`

FlowStorm CLJS debugging is remote debugging: the app/runtime and debugger UI communicate over nREPL/websocket.

---

## ClojureScriptStorm With shadow-cljs

Minimums:

- shadow-cljs >= 2.25.4 for ClojureScript 1.11
- shadow-cljs >= 3.1.1 for ClojureScript 1.12
- FlowStorm >= 3.7.4

`shadow-cljs.edn`:

```clojure
{:deps {:aliases [:1.12-cljs-storm]}
 :nrepl {:port 9000}
 :builds
 {:my-app
  {:devtools {:preloads [flow-storm.storm-preload]
              :http-port 8021}}}}
```

Put `flow-storm.storm-preload` first if there are multiple preloads.

`deps.edn`:

```clojure
{:aliases
 {:1.12-cljs-storm
  {:classpath-overrides {org.clojure/clojurescript nil}
   :extra-deps {thheller/shadow-cljs {:mvn/version "3.3.4"
                                      :exclusions [org.clojure/clojurescript]}
                com.github.flow-storm/clojurescript {:mvn/version "1.12.134-3"}
                com.github.flow-storm/flow-storm-inst {:mvn/version "4.5.9"}}}}}
```

Run the shadow build normally, then start the UI:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port 9000 :repl-type :shadow :build-id :my-app
```

Reload the browser page so the runtime connects to the UI.

---

## ClojureScriptStorm With cljs.main

Start a CLJS REPL with ClojureScriptStorm and a preload:

```bash
clj -Sforce -J-Dcljs.storm.instrumentOnlyPrefixes=cljs.user -Sdeps '{:deps {com.github.flow-storm/clojurescript {:mvn/version "1.12.134-3"} com.github.flow-storm/flow-storm-inst {:mvn/version "4.5.9"}}}' -M -m cljs.main -co '{:preloads [flow-storm.storm-preload]}' --repl
```

Start the UI in another terminal:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger
```

Refresh the browser page so it connects.

---

## Vanilla shadow-cljs

Add the runtime dependency and preload:

```clojure
{:dependencies [[com.github.flow-storm/flow-storm-inst "4.5.9"]]
 :nrepl {:port 9000}
 :builds
 {:my-build-id
  {:devtools {:preloads [flow-storm.preload]}}}}
```

Start shadow:

```bash
npx shadow-cljs watch :my-build-id
```

Start the debugger:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port 9000 :repl-type :shadow :build-id :my-build-id
```

Trace from the CLJS REPL:

```clojure
#rtrace (reduce + (map inc (range 10)))
```

---

## Multiple Builds

Run one debugger per build, each with a unique websocket port:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port 9000 :repl-type :shadow :build-id :my-app :ws-port 7722

clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port 9000 :repl-type :shadow :build-id :my-worker :ws-port 7733
```

Use build-specific preloads that call `flow-storm.runtime.debuggers-api/remote-connect` with the matching `:debugger-ws-port`.

---

## Common CLJS Limitations

- With cljs.main, instrumentation changes from the UI may require restarting the REPL.
- In CLJS, typed simple REPL expressions may not record unless wrapped in an immediately invoked function.
- Outputs supports taps in CLJS; last evals and stdout/stderr capture are Clojure-only.
- React Native often needs `:debugger-host` set to the dev machine IP.
