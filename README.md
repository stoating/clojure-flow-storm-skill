# Clojure FlowStorm Skill

A structured Markdown skill that teaches AI coding agents how to work with FlowStorm - an omniscient time-travel debugger for Clojure and ClojureScript.

Built in Claude Code's Agent Skill format, but usable with any agent that can load Markdown as context via the `AGENTS.md` convention.

## Installation

Claude Code via this plugin marketplace:

```text
/plugin marketplace add stoating/clojure-flow-storm-skill
/plugin install flow-storm@clojure-flow-storm-skill
```

Claude Code via the aggregate marketplace:

```text
/plugin marketplace add stoating/plugins
/plugin install flow-storm@stoating
```

Manual context usage:

```bash
aider --read clojure-flow-storm-skill/flow-storm/SKILL.md
codex --ask-for-approval on-request
```

## Repository Layout

```text
.
.claude-plugin/
  marketplace.json
  plugin.json
AGENTS.md
README.md
flow-storm/
  SKILL.md
  setup.md
  clojurescript.md
  instrumentation.md
  debugging-workflow.md
  remote-and-runtime.md
  anti-patterns.md
```

Only `flow-storm/` contains the skill content. The rest is metadata and cross-agent entry points.
