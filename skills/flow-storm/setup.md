# FlowStorm Setup

## Contents

- Choosing a mode
- ClojureStorm with tools.deps
- ClojureStorm with Leiningen
- Vanilla FlowStorm
- Starting and stopping
- Useful JVM properties

---

## Choosing A Mode

Use ClojureStorm when debugging normal Clojure development code. It replaces `org.clojure/clojure` on the dev classpath with FlowStorm's patched compiler and instruments automatically.

Use vanilla FlowStorm when:

- compiler swapping is not acceptable
- you only want to trace selected forms
- you are in ClojureScript without ClojureScriptStorm
- you are experimenting in a small REPL session

Use `flow-storm-dbg` when the UI runs in the same process. Use `flow-storm-inst` on a runtime process when the UI runs elsewhere.

---

## ClojureStorm With Tools.deps

Project or global alias:

```clojure
{:aliases
 {:1.12-storm
  {:classpath-overrides {org.clojure/clojure nil}
   :extra-deps {com.github.flow-storm/clojure {:mvn/version "1.12.4"}
                com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}}}
```

Start the REPL:

```bash
clj -A:1.12-storm
```

Then evaluate the keyword:

```clojure
:dbg
```

Useful ClojureStorm REPL commands:

| Command | Effect |
|---------|--------|
| `:dbg` | Show the FlowStorm UI |
| `:rec` | Start recording |
| `:stop` | Stop recording |
| `:last` | Jump the UI to the last expression in the current thread |
| `:help` | Print FlowStorm command help |

One-off tutorial REPL:

```bash
clj -Sforce -Sdeps '{:deps {} :aliases {:dev {:classpath-overrides {org.clojure/clojure nil} :extra-deps {com.github.flow-storm/clojure {:mvn/version "1.12.4"} com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}}}' -A:dev
```

---

## ClojureStorm With Leiningen

Profile:

```clojure
{:profiles
 {:1.12-storm
  {:dependencies [[com.github.flow-storm/clojure "1.12.4"]
                  [com.github.flow-storm/flow-storm-dbg "4.5.9"]]
   :exclusions [org.clojure/clojure]}}}
```

Start with the profile added:

```bash
lein with-profile +1.12-storm repl
```

Then evaluate `:dbg`.

For Leiningen versions before 2.11.0, be careful with global dependencies that include official `org.clojure/clojure`.

---

## Vanilla FlowStorm

Dependency:

```clojure
{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}
```

One-off REPL:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}'
```

Start the debugger:

```clojure
(require '[flow-storm.api :as fs-api])

(fs-api/local-connect)
```

Trace and run a form:

```clojure
#rtrace (reduce + (map inc (range 10)))
```

`#rtrace` returns the expression result and records the execution.

---

## Starting And Stopping

In ClojureStorm:

```clojure
:dbg
:rec
:stop
```

In API-based workflows:

```clojure
(require '[flow-storm.api :as fs-api])

(fs-api/local-connect {:theme :dark})
(fs-api/start-recording)
(fs-api/stop-recording)
(fs-api/stop)
```

`local-connect` accepts options such as `:theme`, `:styles`, `:title`, and `:verbose?`.

---

## Useful JVM Properties

```text
-Dflowstorm.startRecording=false
-Dflowstorm.theme=dark
-Dflowstorm.title=FlowStormMainDebugger
-Dflowstorm.styles=/path/to/styles.css
-Dflowstorm.fileEditorCommand=code --goto <<FILE>>:<<LINE>>
-Dflowstorm.threadTraceLimit=1000
-Dflowstorm.throwOnLimit=true
-Dflowstorm.heapLimit=1000
-Dflowstorm.callTreeUpdate=false
```

Use `flowstorm.startRecording=false` for noisy apps and enable recording just before the action under investigation.
