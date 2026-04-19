# FlowStorm Debugging Workflow

## Contents

- Flows and recordings
- Code stepping
- Search
- Outputs and taps
- Data Windows
- Thread breakpoints
- Programmable debugging

---

## Flows And Recordings

A flow groups recorded execution activity. Use separate flows for separate runs so behavior before and after a change can be compared.

Typical loop:

1. Clear old recordings.
2. Start recording.
3. Trigger the action or run the expression.
4. Stop recording.
5. Explore the thread timeline, call tree, values, outputs, and searches.

Keep recording paused for noisy apps and enable it only around the action under investigation.

---

## Code Stepping

The Code tool steps through recorded expressions, not live mutable memory. This fits Clojure code well because most recorded values are immutable references.

Useful stepping modes:

- first/last recording
- previous/next step
- previous/next same source coordinate
- function call stack navigation
- jump to the first recording for a source coordinate
- jump backward/forward to a selected coordinate

If a source expression is not highlighted, either it was not recorded, it belongs to a different function frame, or the code was not instrumented at the right level.

---

## Search

Search over recorded expressions by:

- printed representation
- value selected in a Data Window
- predicate

Use search when you do not know where a value originated. For tap markers that should be searchable, tap an expression value such as `(tap> (str :my-mark))` instead of a literal keyword.

---

## Outputs And Taps

For full Outputs support, use nREPL middleware:

```clojure
flow-storm.nrepl.middleware/wrap-flow-storm
```

Outputs can show:

- last evaluation results
- tapped values
- `*out*` and `*err*`

FlowStorm adds a tap on startup. Tapped values appear in the Taps list, and values that were also recorded in Flows can be searched back into the execution timeline.

Reader tags:

```clojure
#tap (+ 1 2)
#tap-stack-trace nil
```

---

## Data Windows

Data Windows inspect and navigate values. They support:

- nested key/value navigation
- metadata navigation
- multiple visualizers
- lazy and infinite sequence paging
- defining the current sub-value into the REPL
- realtime visual updates
- `clojure.datafy/datafy` and `clojure.datafy/nav`

Create or update a Data Window from code:

```clojure
(require '[flow-storm.api :as fs-api])

(fs-api/data-window-push-val :my-window {:status :starting} "status")
(fs-api/data-window-val-update :my-window {:status :ready})
```

Data Windows show the result of `datafy`. For maps/vectors with navigable keys, FlowStorm provides navigation affordances for `nav`.

---

## Thread Breakpoints

Thread breakpoints pause threads at selected functions, useful when recording everything is too expensive or when thread interleavings matter.

Install programmatically:

```clojure
(require '[flow-storm.api :as fs-api])

(fs-api/break-at 'my.app/suspicious-fn)
(fs-api/remove-break 'my.app/suspicious-fn)
(fs-api/clear-breaks)
(fs-api/unblock-all-threads)
```

When a thread hits a breakpoint and recording is on, FlowStorm blocks it and exposes blocked threads in the UI. Pause recording before unblocking if you do not want the resumed work recorded.

---

## Programmable Debugging

For advanced analysis from the REPL, require the indexes API:

```clojure
(require '[flow-storm.runtime.indexes.api :as ia])
```

Use it to inspect timelines, forms, function calls, and multi-thread timelines. This is useful when a question is easier to answer with Clojure over recorded data than by clicking through the UI.
