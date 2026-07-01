# Learning Paths

## Automation-First Path

This is the default learning path for the repository.

### Stage 1: Shell And Workspace Foundations

Goal: become comfortable moving through the terminal, managing files, and using Git safely.

Recommended order:

1. topics/terminal/README.md
2. topics/environment-variables/README.md
3. topics/git/README.md
4. languages/bash/beginner/README.md

### Stage 2: Scripting Fluency

Goal: write safe, repeatable local automation.

Recommended order:

1. languages/bash/intermediate/README.md
2. languages/python/beginner/README.md

### Stage 3: Structured Data And Integration

Goal: automate workflows involving configuration, APIs, and structured input/output.

Recommended order:

1. topics/json/README.md
2. topics/http-and-apis/README.md
3. topics/package-managers/README.md
4. languages/python/intermediate/README.md

### Stage 4: Quality And Reliability

Goal: test, debug, and harden automation before it is shared or scheduled.

Recommended order:

1. topics/debugging/README.md
2. topics/testing/README.md
3. topics/security-basics/README.md

### Stage 5: Delivery And Operations

Goal: ship automation in reproducible environments and CI systems.

Recommended order:

1. topics/docker-basics/README.md
2. topics/github/README.md
3. topics/gitlab/README.md when GitLab CI is the learner's delivery platform
4. languages/bash/advanced/README.md
5. languages/python/advanced/README.md

## Secondary Paths

### Agentic Automation Optional Path [PLANNED]

Use this after Stage 3 when the learner can already work with scripts, JSON, HTTP APIs, and package-managed tools. Learners who want more reliability practice first should complete Stage 4 before starting this path. Stage 5 delivery and operations are useful later but are not required before beginning agentic automation.

Recommended order:

1. [PLANNED] Agentic foundations: understand what an agent is, where agent boundaries belong, and why one agent should have one clear job.
2. [PLANNED] `topics/agent-building/README.md`: design specialized agents, orchestration flow, state handling, and decision patterns.
3. [PLANNED] `topics/mcp-integration/README.md`: connect agents to MCP servers, tool discovery, and local tool surfaces.
4. [PLANNED] `topics/agent-tool-integration/README.md`: define tool schemas, validate parameters, handle tool errors, and manage handoffs.
5. [PLANNED] `topics/ide-agent-workflows/README.md`: invoke and coordinate agent workflows in VS Code, Codex, and Claude Code.

Milestone outcome: build a specialized automation agent that uses mock MCP tools, validates tool inputs, handles failures, and can be invoked from VS Code, Codex, or Claude Code.

### Language-First Path

Use this when a learner needs one language deeply before widening into automation topics.

- Pick one language track.
- Pair it with terminal, git, testing, debugging, and environment variables.
- Rejoin the automation-first path by Stage 3.

### Tooling-First Path

Use this when a learner already programs but lacks delivery and automation tooling.

Recommended emphasis:

1. terminal
2. git
3. bash intermediate and advanced
4. docker basics
5. github or gitlab delivery workflows
6. debugging and testing

## Path Rules

1. Prefer README files as entry points before deep lesson runs.
2. Use this file to justify priority ordering in TODO.md.
3. Update this file when the director changes sequencing or introduces milestone projects.
4. Do not treat every topic as required for the automation-first learner.

See [CURRICULUM_MAP.md](CURRICULUM_MAP.md) for the stage-to-track mapping that supports these paths.
