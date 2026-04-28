# FlowStorm Remote And Runtime

## Contents

- Remote Clojure over nREPL
- SSH tunnels
- Split runtime and UI processes
- Docker
- nREPL middleware
- Editor commands

---

## Remote Clojure Over nREPL

The debuggee process needs an nREPL server and FlowStorm runtime dependency. Use `flow-storm-inst` when the UI is not in the same process:

```clojure
{:aliases
 {:runtime-storm
  {:classpath-overrides {org.clojure/clojure nil}
   :extra-deps {com.github.flow-storm/clojure {:mvn/version "1.12.4"}
                com.github.flow-storm/flow-storm-inst {:mvn/version "4.5.9"}}}}}
```

Start the UI from the dev machine:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port 9000
```

Use full `flow-storm-dbg` on the runtime side only when local UI libraries on the debuggee are acceptable.

---

## SSH Tunnels

If the remote app exposes nREPL on `localhost:9000`, create a tunnel:

```bash
ssh -L 9000:localhost:9000 -R 7722:localhost:7722 my-debuggee-box.com
```

Then start the UI locally:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port 9000
```

Without a tunnel, specify hosts and websocket port:

```bash
clj -Sforce -Sdeps '{:deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}}' -X flow-storm.debugger.main/start-debugger :port NREPL-PORT :runtime-host '"APP_HOST"' :debugger-host '"DEV_HOST"' :ws-port WS_SERVER_PORT
```

---

## Split Runtime And UI Processes

Use separate aliases:

```clojure
{:aliases
 {:runtime-storm
  {:classpath-overrides {org.clojure/clojure nil}
   :extra-deps {com.github.flow-storm/clojure {:mvn/version "1.12.4"}
                com.github.flow-storm/flow-storm-inst {:mvn/version "4.5.9"}}}

  :ui-storm
  {:extra-deps {com.github.flow-storm/flow-storm-dbg {:mvn/version "4.5.9"}}
   :exec-fn flow-storm.debugger.main/start-debugger
   :exec-args {:port 7888}}}}
```

Start the app with `:runtime-storm`, then run:

```bash
clj -X:ui-storm
```

---

## nREPL Middleware

For Outputs, last evals, stdout/stderr capture, and richer REPL integration, add:

```clojure
flow-storm.nrepl.middleware/wrap-flow-storm
```

VS Code example:

```json
{
  "name": "flowstorm",
  "projectType": "deps.edn",
  "extraNReplMiddleware": ["flow-storm.nrepl.middleware/wrap-flow-storm"],
  "afterCLJReplJackInCode": "((requiring-resolve 'flow-storm.storm-api/start-debugger))",
  "cljsType": "none",
  "menuSelections": {
    "cljAliases": ["flowstorm"]
  }
}
```

For CIDER, add the middleware to `cider-jack-in-nrepl-middlewares` or project `.dir-locals.el`.

---

## Editor Commands

Configure file opening from the UI:

```text
-Dflowstorm.fileEditorCommand=code --goto <<FILE>>:<<LINE>>
-Dflowstorm.fileEditorCommand=idea --line <<LINE>> <<FILE>>
-Dflowstorm.fileEditorCommand=emacsclient -n +<<LINE>>:0 <<FILE>>
-Dflowstorm.fileEditorCommand=vim +<<LINE>> <<FILE>>
```

For jar source:

```text
-Dflowstorm.jarEditorCommand=emacsclient -n +<<LINE>>:0 <<JAR>>/<<FILE>>
```

Use absolute editor commands when running from shells with restricted PATH.
