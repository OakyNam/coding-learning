# Coding Learning

This repository is an automation-first learning space for developers.

It is designed for people who want to get better at:

- scripting repetitive work
- automating local developer workflows
- working with terminals, files, JSON, and HTTP APIs
- testing and debugging automation safely
- shipping automation through containers and CI/CD platforms

The curriculum uses language tracks and topic tracks together. Languages provide implementation tools. Topics provide cross-cutting skills that make automation work reliable, portable, and maintainable.

## Start Here

- Read [CURRICULUM_STRATEGY.md](CURRICULUM_STRATEGY.md) for curriculum purpose, scope tiers, and priority clusters.
- Read [LEARNING_PATHS.md](LEARNING_PATHS.md) for recommended routes through the material.
- Read [CURRICULUM_MAP.md](CURRICULUM_MAP.md) for the stage-to-track map that connects strategy to concrete repo paths.
- Read [TODO.md](TODO.md) for the current execution inventory.

## Recommended Path

For most learners, start with the automation-first path:

1. [topics/terminal/README.md](topics/terminal/README.md)
2. [topics/environment-variables/README.md](topics/environment-variables/README.md)
3. [topics/git/README.md](topics/git/README.md)
4. [languages/bash/beginner/README.md](languages/bash/beginner/README.md)
5. [languages/bash/intermediate/README.md](languages/bash/intermediate/README.md)
6. [languages/python/beginner/README.md](languages/python/beginner/README.md)
7. [topics/json/README.md](topics/json/README.md)
8. [topics/http-and-apis/README.md](topics/http-and-apis/README.md)
9. [topics/package-managers/README.md](topics/package-managers/README.md)
10. [languages/python/intermediate/README.md](languages/python/intermediate/README.md)
11. [topics/debugging/README.md](topics/debugging/README.md)
12. [topics/testing/README.md](topics/testing/README.md)
13. [topics/security-basics/README.md](topics/security-basics/README.md)
14. [topics/docker-basics/README.md](topics/docker-basics/README.md)
15. [topics/github/README.md](topics/github/README.md) or [topics/gitlab/README.md](topics/gitlab/README.md), depending on the CI/CD platform you use

See [LEARNING_PATHS.md](LEARNING_PATHS.md) for alternate language-first and tooling-first routes.

## Repository Shape

### Languages

Language tracks teach implementation skills at beginner, intermediate, and advanced levels.

- [languages/bash/README.md](languages/bash/README.md)
- [languages/c/README.md](languages/c/README.md)
- [languages/csharp/README.md](languages/csharp/README.md)
- [languages/java/README.md](languages/java/README.md)
- [languages/javascript/README.md](languages/javascript/README.md)
- [languages/perl/README.md](languages/perl/README.md)
- [languages/php/README.md](languages/php/README.md)
- [languages/python/README.md](languages/python/README.md)

### Topics

Topic tracks teach the supporting knowledge that automation work depends on.

- [topics/README.md](topics/README.md)

## How To Use The Material

1. Start from a README file before diving into individual lesson files.
2. Follow one path long enough to build momentum instead of sampling everything at once.
3. Complete exercises and worked answers as part of the lesson flow.
4. Use the strategy and path documents when deciding what to study next or what to improve in the curriculum.

## Curriculum Governance

The curriculum is improved through a strategy layer and an execution layer.

- The director owns strategy, learning paths, and rewrite priorities.
- The assistant director reviews strategy changes before they become active.
- The supervisor and file-level agents execute approved changes through the rolling rewrite workflow.

Use [.codex/workflows/curriculum-strategy-refresh.md](.codex/workflows/curriculum-strategy-refresh.md) for macro strategy refreshes before resuming file-level execution.

The implementation workflow lives in [.codex/workflows/curriculum-audit-rewrite.md](.codex/workflows/curriculum-audit-rewrite.md).
