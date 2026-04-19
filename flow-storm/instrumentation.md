# FlowStorm Instrumentation

## Contents

- ClojureStorm instrumentation
- Startup prefixes and skips
- Browser instrumentation
- Vanilla `#trace` and `#rtrace`
- API instrumentation
- Uninstrumenting

---

## ClojureStorm Instrumentation

ClojureStorm and ClojureScriptStorm instrument newly compiled code. Changing instrumentation settings does not rewrite already-loaded code unless namespaces are reloaded.

By default, FlowStorm can infer top-level project namespaces from source folders on the classpath. Prefer explicit prefixes for large or unusual projects.

```text
-Dclojure.storm.instrumentOnlyPrefixes=my-app,my-lib
-Dclojure.storm.instrumentSkipPrefixes=my-app.too-heavy,my-lib.uninteresting
-Dclojure.storm.instrumentSkipRegex=.*test.*
```

ClojureScriptStorm equivalents:

```text
-Dcljs.storm.instrumentOnlyPrefixes=my-app,my-lib
-Dcljs.storm.instrumentSkipPrefixes=my-app.too-heavy,my-lib.uninteresting
```

Use `instrumentOnlyPrefixes` for what should be traced and skip prefixes/regex for hot or irrelevant code.

---

## Browser Instrumentation

The Browser tool can add/remove instrumentation while the REPL is running.

Common actions:

- instrument a var
- recursively instrument a var and referred vars
- instrument selected namespaces with `:light`
- instrument selected namespaces with `:full`
- disable or delete existing instrumentation entries
- fully instrument a form from the Code view after starting with light instrumentation

Light instrumentation records function arguments and return values. Full instrumentation records expressions and bindings.

When FlowStorm offers to reload affected namespaces after changing prefixes, accept it for normal REPL workflows. Otherwise manually reload:

```clojure
(require 'my.selected.namespace :reload)
```

Use callbacks when namespace reload should stop and restart the system:

```clojure
(flow-storm.api/set-before-reload-callback! #(println "Before reloading"))
(flow-storm.api/set-after-reload-callback!  #(println "After reloading"))
```

---

## Vanilla `#trace` And `#rtrace`

`#trace` instruments a top-level form that does not run immediately:

```clojure
#trace
(defn sum [a b]
  (+ a b))
```

`#rtrace` instruments and runs an expression:

```clojure
#rtrace (->> (range)
             (filter odd?)
             (take 10)
             (reduce +))
```

Use `#rtrace` for REPL expressions. Use `#trace` for definitions such as `defn`, `defmethod`, `extend-type`, and similar top-level forms.

`#tap` taps and returns a value:

```clojure
(+ 1 2 #tap (* 3 4))
```

`#tap-stack-trace` taps the current stack trace.

---

## API Instrumentation

```clojure
(require '[flow-storm.api :as fs-api])

(fs-api/instrument-var-clj 'my.app/foo)
(fs-api/uninstrument-var-clj 'my.app/foo)

(fs-api/instrument-namespaces-clj #{"my.app" "my.lib"}
                                  {:excluding-ns #{"my.app.too-heavy"}
                                   :disable #{:binding :anonymous-fn}})

(fs-api/uninstrument-namespaces-clj #{"my.app"})
```

Options for namespace instrumentation:

| Option | Use |
|--------|-----|
| `:excluding-ns` | Exact namespaces to exclude |
| `:disable` | Disable `:expr`, `:binding`, or `:anonymous-fn` traces |
| `:verbose?` | Print more instrumentation logging |

For ClojureScript from a shadow Clojure REPL:

```clojure
(fs-api/instrument-var-cljs 'my.app/foo {:build-id :my-app})
(fs-api/instrument-namespaces-cljs #{"my.app"} {:build-id :my-app})
```

---

## What Cannot Be Instrumented Reliably

- Very large forms can exceed JVM method size limits after instrumentation.
- Functions that call `recur` without a `loop`.
- Functions that return recursive lazy sequences.
- Code inside macros that do not preserve source metadata may not map cleanly to source.
- `clojure.core` and FlowStorm internals should not be instrumented because they can recurse into tracing machinery.

If a namespace is too noisy, start with light instrumentation and fully instrument only the form or function of interest.
