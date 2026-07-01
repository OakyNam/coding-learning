# Curriculum Map

## Purpose

This file maps the main curriculum surfaces to the automation-first learner journey.

Use it to answer three questions:

1. What should a learner study next?
2. Which tracks are core versus supporting?
3. Which parts of the repository should the director prioritize for improvement?

## Stage Map

### Stage 1: Shell And Workspace Foundations

Primary outcome: move confidently through terminals, files, paths, and repositories.

Core tracks:

- [topics/terminal/README.md](topics/terminal/README.md)
- [topics/environment-variables/README.md](topics/environment-variables/README.md)
- [topics/git/README.md](topics/git/README.md)
- [languages/bash/beginner/README.md](languages/bash/beginner/README.md)

### Stage 2: Scripting Fluency

Primary outcome: write safe, reusable local automation.

Core tracks:

- [languages/bash/intermediate/README.md](languages/bash/intermediate/README.md)
- [languages/python/beginner/README.md](languages/python/beginner/README.md)

### Stage 3: Structured Data And Integration

Primary outcome: automate workflows involving JSON, APIs, and package-managed tools.

Core tracks:

- [topics/json/README.md](topics/json/README.md)
- [topics/http-and-apis/README.md](topics/http-and-apis/README.md)
- [topics/package-managers/README.md](topics/package-managers/README.md)
- [languages/python/intermediate/README.md](languages/python/intermediate/README.md)

### Stage 4: Quality And Reliability

Primary outcome: test, debug, and harden automation so it behaves predictably.

Core tracks:

- [topics/debugging/README.md](topics/debugging/README.md)
- [topics/testing/README.md](topics/testing/README.md)
- [topics/security-basics/README.md](topics/security-basics/README.md)

### Stage 5: Delivery And Operations

Primary outcome: package and run automation in repeatable delivery environments.

Core tracks:

- [topics/docker-basics/README.md](topics/docker-basics/README.md)
- [topics/github/README.md](topics/github/README.md)
- [topics/gitlab/README.md](topics/gitlab/README.md) when GitLab CI is the delivery platform
- [languages/bash/advanced/README.md](languages/bash/advanced/README.md)
- [languages/python/advanced/README.md](languages/python/advanced/README.md)

## Scope Classification

### Core For Automation

- terminal
- environment variables
- git
- github
- gitlab as an alternate CI/CD platform
- package managers
- docker basics
- debugging
- testing
- security basics
- bash
- python
- json
- http and apis

### Supporting For Automation

- markdown
- regex
- sql
- oop
- web basics
- algorithms
- data structures

### Planned Expansion

- infrastructure as code
- cloud automation
- configuration management
- secrets management
- observability
- event-driven automation
- [PLANNED] agentic automation
- [PLANNED] agent building
- [PLANNED] mcp integration
- [PLANNED] agent-tool integration
- [PLANNED] IDE agent workflows for VS Code, Codex, and Claude Code

## Optional Agentic Automation Branch [PLANNED]

This branch is learner-facing expansion work. It may start after Stage 3 for learners who already have basic debugging discipline, or after Stage 4 for learners who want more reliability practice first. Stage 5 delivery and operations are useful later but are not prerequisites for this branch.

### Agentic Branch Entry Point

Primary outcome: build from structured data, APIs, scripts, and testing habits into agent-driven automation.

Prerequisite tracks:

- Stage 3: Structured Data And Integration
- Stage 4: Quality And Reliability, recommended before larger multi-tool agent work

### Planned Agentic Tracks

- [PLANNED] `topics/agent-building/README.md`: specialized agents, one-job boundaries, orchestration patterns, state handling, and decision flow
- [PLANNED] `topics/mcp-integration/README.md`: MCP servers, tool discovery, local tool surfaces, and protocol-shaped handoffs
- [PLANNED] `topics/agent-tool-integration/README.md`: tool schemas, parameter validation, handoff contracts, and error recovery
- [PLANNED] `topics/ide-agent-workflows/README.md`: VS Code, Codex, and Claude Code workflows for invoking and coordinating agents

### Optional Agentic Milestone

Build a specialized automation agent that uses mock MCP tools, validates tool inputs, handles tool errors, and can be invoked from VS Code, Codex, or Claude Code.

## Director Usage

The director should use this file to:

- connect strategy to concrete repo paths
- justify TODO prioritization by stage or scope tier
- show where a rewrite batch fits into the larger learning path
- identify missing bridges between tracks

The assistant director should review this file when stage ordering, scope classification, or path mapping changes.
