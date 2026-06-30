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
