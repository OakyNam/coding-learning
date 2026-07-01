# Curriculum Strategy

## Purpose

This repository teaches developers how to automate real work.

The curriculum is optimized for developers who need to:

- automate local tasks and repetitive workflows
- script developer tooling and environment setup
- work with files, APIs, JSON, and command-line tools
- build reliable CI and delivery automation
- grow from local scripting into broader platform and operations automation

Language depth matters, but it is in service of automation outcomes rather than breadth for its own sake.

## Primary Learner

The primary learner is a developer who wants to become effective at automation through scripting, tooling, integration work, testing, debugging, and delivery practices.

Typical learner goals:

- write safe Bash and Python automation
- understand terminals, paths, environment variables, and processes
- automate HTTP and JSON workflows
- validate and test automation work
- package, run, and ship automation through CI/CD

## Repo Evidence Snapshot

The current repository supports this strategy with:

- core automation language tracks for Bash and Python
- additional language tracks for C, C#, Java, JavaScript, Perl, and PHP
- topic tracks for terminals, environment variables, Git, GitHub, GitLab, package managers, Docker, debugging, testing, security, JSON, HTTP APIs, SQL, regex, Markdown, OOP, web basics, algorithms, and data structures
- newer advanced language lessons for defensive input validation, structured data interchange or serialization, and logging or observability

## Scope Tiers

### Core

These topics directly support the automation-first path and should receive the highest rewrite and navigation priority.

- terminal
- git
- github
- gitlab, when the learner's delivery environment uses GitLab CI
- environment variables
- package managers
- docker basics
- debugging
- testing
- security basics
- bash
- python
- json
- http and apis

### Supporting

These topics improve developer judgment and code quality, but are not the shortest path to automation fluency.

- regex
- sql
- oop
- web basics
- algorithms
- data structures
- markdown

### Expansion

These should shape future roadmap planning once the core path is coherent and well-maintained.

- infrastructure as code
- cloud automation
- configuration management
- secrets management
- observability for automation
- event-driven and webhook automation
- [PLANNED] agentic automation
- [PLANNED] agent building
- [PLANNED] mcp integration
- [PLANNED] agent-tool integration
- [PLANNED] IDE agent workflows for VS Code, Codex, and Claude Code

## Curriculum Rules

1. Prioritize learner outcomes over equal treatment of every topic.
2. Prefer connected learning paths over isolated lesson islands.
3. Keep the rewrite queue focused on a small number of high-value clusters.
4. Use strategy artifacts to explain priorities instead of carrying broad context in every file-level prompt.
5. Favor additive navigation and clearer sequencing before large folder moves.

## Priority Clusters

### Cluster 1: Automation Foundations

- terminal
- environment variables
- git
- bash beginner and intermediate
- python beginner and intermediate

### Cluster 2: Automation Integration

- json
- http and apis
- package managers
- debugging
- testing

### Cluster 3: Automation Delivery

- docker basics
- github actions and related github lessons
- gitlab ci and runner lessons as an alternate delivery path
- deployment-oriented bash and python content
- advanced validation, structured data, logging, and observability lessons
- security basics

### Cluster 4: Expansion Agentic Automation [PLANNED]

This optional expansion cluster branches after Stage 3 or Stage 4 once learners can work with structured data, APIs, scripts, and basic reliability practices. It is not required before delivery and operations work.

- agent building: specialized agents, one-job boundaries, orchestration patterns, state handling, and decision flow
- mcp integration: MCP servers, tool discovery, local tool surfaces, and protocol-shaped handoffs
- agent-tool integration: tool schemas, parameter validation, handoff contracts, and error recovery
- IDE agent workflows: invoking and coordinating agent workflows in VS Code, Codex, and Claude Code

## Milestone Outcomes

The curriculum should eventually guide learners to these outcomes:

1. Build a local automation script that validates input, writes output, and fails clearly.
2. Build a Python or Bash tool that reads files or APIs and produces structured results.
3. Test and debug an automation workflow with predictable inputs and outputs.
4. Run automation through CI with clear logs and artifacts.
5. Package or containerize automation for repeatable execution.
6. Optionally build a specialized automation agent that uses mock MCP tools, validates tool inputs, and can be invoked from VS Code, Codex, or Claude Code.

## Director Responsibilities

The curriculum director should use this file to:

- refine purpose and learner assumptions
- classify priority clusters
- explain major sequencing decisions
- justify changes to TODO generation or navigation

See [CURRICULUM_MAP.md](CURRICULUM_MAP.md) for the path-to-repo mapping that supports those decisions.

The assistant director should review this file whenever strategy materially changes.
