# Agent instructions

This repository contains the **`flow-storm`** skill - a structured set of Markdown files that teach an AI coding agent how to work with FlowStorm, the Clojure/ClojureScript omniscient debugger.

**Any agent that understands this `AGENTS.md` convention should:**

1. Treat `flow-storm/SKILL.md` as the entry point - it contains a decision table pointing to the right reference file for the task.
2. Load reference files on demand based on that table:
   - `setup.md` - ClojureStorm, vanilla FlowStorm, deps.edn, Leiningen, UI startup, JVM properties
   - `clojurescript.md` - ClojureScriptStorm, shadow-cljs, cljs.main, vanilla CLJS, multiple builds
   - `instrumentation.md` - prefixes, skips, Browser tool, `#trace`, `#rtrace`, var/ns instrumentation
   - `debugging-workflow.md` - flows, stepping, search, Outputs, taps, Data Windows, thread breakpoints
   - `remote-and-runtime.md` - remote nREPL, SSH tunnels, split runtime/UI, middleware, editor commands
   - `anti-patterns.md` - trace volume, heap limits, mutable values, macro metadata, remote setup mistakes
3. Prefer ClojureStorm/ClojureScriptStorm for normal development setup and vanilla FlowStorm for manual targeted tracing.
4. Warn users to bound instrumentation and recording before tracing noisy apps, loops, streams, UI events, or large dependency trees.

This file follows the [agents.md](https://agents.md/) convention and is honored by OpenAI Codex CLI, Cursor, Aider, Zed, Amp, Gemini CLI, Google Jules, Windsurf, Factory, RooCode, and many others.

For Claude Code, the richer native format is `.claude-plugin/` + `flow-storm/SKILL.md`.
