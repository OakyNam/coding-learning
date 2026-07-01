# Curriculum Rewrite TODO

Strategy references:

- `CURRICULUM_STRATEGY.md` is the source of truth for curriculum purpose, scope tiers, and priority clusters.
- `LEARNING_PATHS.md` is the source of truth for learner sequencing and path intent.
- `CURRICULUM_MAP.md` is the source of truth for stage-to-track mapping and rewrite-cluster context.

Status values: `needs-audit`, `audit-dispatched`, `audit-reviewed`, `rewrite-dispatched`, `rewrite-reviewed`, `compatibility-review-needed`, `compatibility-review-dispatched`, `compatibility-reviewed`, `strategy-review-needed`, `strategy-pass`, `complete`.

Priority tags: `core`, `supporting`, `expansion`.

Path tags: `automation-first`, `language-first`, `tooling-first`, `agentic-automation`.

Cluster tags:

- `automation-foundations`
- `automation-integration`
- `automation-delivery`
- `supporting-systems`
- `supporting-web-data`
- `expansion-platform`
- `expansion-agentic`

Row format:

- Required fields: `type`, `owner`, `status`, `sources`
- Director metadata fields when available: `priority`, `path`, `cluster`
- Supervisors may update `owner` and `status` without changing approved `priority`, `path`, or `cluster` values.

Execution rule: the director generates or reprioritizes this file, the assistant director verifies strategic changes, and the supervisor updates operational state while preserving approved priority order.

This inventory is generated from the repo file list. Lesson files need exercises and worked answers; README/workflow/support files need clarity and accurate navigation.

Compatibility baseline: files accepted before this gate must be re-reviewed for Windows 10/11 PowerShell and macOS Apple Silicon (`arm64`, `zsh`) instructions. Mermaid diagrams may be used when they clarify an accurate user-to-code data flow and render in GitHub and the VS Code Mermaid extension.

## Director-Approved Execution Queue

Use this queue before the raw generated inventory order below. Keep the two-file rolling gate: finish the active file first, then pull at most two `needs-audit` files from this queue.

Active file already in the gate:

- no strategy-owned doc file is currently active in the gate

Next automation-first batch:

1. `topics\terminal\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations
2. `topics\terminal\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations
3. `topics\environment-variables\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations
4. `topics\environment-variables\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations
5. `topics\git\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations
6. `topics\git\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations
7. `languages\python\beginner\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations
8. `languages\python\beginner\01_setup_and_install.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations

Next integration batch after foundations:

1. `topics\json\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-integration
2. `topics\json\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-integration
3. `topics\http-and-apis\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-integration
4. `topics\http-and-apis\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-integration
5. `topics\package-managers\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-integration
6. `topics\package-managers\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-integration

Metadata policy: rows outside the director queue remain generated inventory until they are pulled into an approved batch or classified by a later strategy pass.

## Director-Approved Agentic Automation Backlog (Strategy Review Needed)

Scope guardrails:

- planning and navigation only; no lesson drafting in this batch
- mark all new learner-facing topics as planned until writer/auditor implementation creates them
- keep the batch bounded to strategy docs and planned README entry points

Assistant-director gate statement:

- every item below is `strategy-review-needed` until assistant-director PASS
- supervisor execution may proceed only after PASS and TODO status promotion to `strategy-pass`

Status values for this subsection:

- `strategy-review-needed`
- `strategy-pass`

Doc-writer batch metadata format:

- Required fields: `type`, `owner`, `status`, `sources`, `priority`, `path`, `cluster`, `governance`

Agentic automation doc-writer batch:

1. `CURRICULUM_STRATEGY.md` | type: strategy-doc | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: strategy-source-of-truth,planned-topic-boundaries,optional-milestone | owner: unassigned | status: strategy-review-needed | sources: repo-evidence-required
2. `LEARNING_PATHS.md` | type: strategy-doc | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: optional-branch-after-stage-3-or-4,no-stage-5-prerequisite | owner: unassigned | status: strategy-review-needed | sources: repo-evidence-required
3. `CURRICULUM_MAP.md` | type: strategy-doc | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: stage-map-branch,planned-topic-boundaries | owner: unassigned | status: strategy-review-needed | sources: repo-evidence-required
4. `README.md` | type: strategy-doc | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: learner-facing-entrypoint,no-standalone-governance-section | owner: unassigned | status: strategy-review-needed | sources: repo-evidence-required
5. `TODO.md` | type: strategy-doc | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: metadata-complete-doc-writer-batch | owner: unassigned | status: strategy-review-needed | sources: repo-evidence-required
6. `topics\agent-building\README.md` [PLANNED] | type: readme | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: one-job-agent-boundaries,orchestration-patterns,state-and-decision-flow | owner: doc-writer | status: strategy-review-needed | sources: planned-topic
7. `topics\mcp-integration\README.md` [PLANNED] | type: readme | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: mcp-servers,tool-discovery,local-tool-surfaces | owner: doc-writer | status: strategy-review-needed | sources: planned-topic
8. `topics\agent-tool-integration\README.md` [PLANNED] | type: readme | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: tool-schemas,parameter-validation,handoff-contracts,error-recovery | owner: doc-writer | status: strategy-review-needed | sources: planned-topic
9. `topics\ide-agent-workflows\README.md` [PLANNED] | type: readme | priority: expansion | path: agentic-automation | cluster: expansion-agentic | governance: vscode,codex,claude-code-agent-workflows | owner: doc-writer | status: strategy-review-needed | sources: planned-topic

Doc-writer handoff note:

- Execute in small batches through the approved strategy + assistant-director gate.
- Keep TODO updates metadata-consistent with existing `priority`, `path`, and `cluster` fields when new rows are added.

## Generated Inventory

- [x] `languages\bash\advanced\01_strict_mode.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\02_traps_and_signals.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\03_portable_shell_scripts.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\04_performance.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\05_parallel_work.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\06_advanced_text_pipelines.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\07_security_for_scripts.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\08_ci_automation.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\09_deployment_scripts.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\10_script_architecture.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\advanced\README.md` | type: readme | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\01_setup_and_shell_basics.md` | type: lesson | owner: review-agent-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\02_commands_files_and_directories.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\03_variables_and_quoting.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\04_input_and_output.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\05_conditionals.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\06_loops.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\07_arrays.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\08_functions.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\09_scripts_and_permissions.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\10_path_and_reusable_commands.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\beginner\README.md` | type: readme | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\01_pipes_and_redirection.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\02_exit_codes_and_status_checks.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\03_arguments_and_flags.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\04_text_processing_basics.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\05_grep_sed_and_awk.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\06_environment_and_configuration.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\07_process_control.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\08_cron_and_scheduling.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\09_robust_scripts.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\10_project_automation_scripts.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\intermediate\README.md` | type: readme | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\bash\README.md` | type: readme | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\c\advanced\01_memory_layout.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\c\advanced\02_undefined_behavior.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\c\advanced\03_function_pointers.md` | type: lesson | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [x] `languages\c\advanced\04_data_structures_in_c.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: coordinator-reviewed-by-codex | status: complete | sources: official-docs-reviewed
- [ ] `languages\c\advanced\05_concurrency_basics.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\06_static_and_dynamic_libraries.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\07_profiling_and_optimization.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\08_portability.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\09_secure_c_patterns.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\10_systems_project_design.md` | type: lesson | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\README.md` | type: readme | priority: supporting | path: language-first | cluster: supporting-systems | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\01_setup_and_install.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\02_project_and_file_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\03_variables_and_data_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\04_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\05_control_flow.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\06_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\07_arrays_and_collections.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\08_structs_and_objects.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\09_functions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\10_headers_and_libraries.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\beginner\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\01_pointers.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\02_dynamic_memory_with_malloc_and_free.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\03_strings_in_depth.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\04_structs_and_pointers.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\05_header_source_organization.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\06_file_io.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\07_error_handling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\08_build_tools_and_makefiles.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\09_debugging_with_a_debugger.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\10_testing_c_programs.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\01_generics_in_depth.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\02_reflection_and_attributes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\03_advanced_linq.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\04_performance_and_memory.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\05_concurrency_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\06_middleware_pipelines.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\07_configuration_and_logging.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\08_packaging_libraries.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\09_cloud_deployment.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\10_architecture_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\01_setup_and_install.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\02_project_and_file_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\03_variables_and_data_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\04_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\05_control_flow.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\06_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\07_arrays_lists_and_collections.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\08_dictionaries_and_objects.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\09_methods.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\10_namespaces_and_packages.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\beginner\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\01_classes_records_and_structs.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\02_interfaces.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\03_collections_and_linq.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\04_exceptions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\05_file_io.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\06_nullable_reference_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\07_unit_testing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\08_async_and_tasks.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\09_dependency_injection.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\10_building_apis_with_asp_net_core.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\01_streams.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\02_advanced_generics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\03_concurrency.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\04_jvm_memory_basics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\05_performance_profiling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\06_reflection_and_annotations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\07_dependency_injection.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\08_spring_overview.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\09_packaging_and_deployment.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\10_architecture_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\01_setup_and_install.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\02_project_and_file_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\03_variables_and_data_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\04_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\05_control_flow.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\06_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\07_arrays_lists_and_collections.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\08_maps_and_objects.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\09_methods.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\10_packages_and_imports.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\beginner\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\01_classes_and_constructors.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\02_encapsulation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\03_inheritance_and_interfaces.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\04_collections_framework.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\05_exceptions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\06_file_io.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\07_generics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\08_unit_testing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\09_maven_or_gradle.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\10_building_http_apis.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\01_closures.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\02_event_loop.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\03_advanced_async_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\04_modules_and_bundling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\05_typescript_overview.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\06_performance.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\07_security_in_web_apps.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\08_testing_strategy.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\09_node_js_services.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\10_deployment.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\01_setup_and_install.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\02_project_and_file_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\03_variables_and_data_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\04_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\05_control_flow.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\06_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\07_arrays_and_collections.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\08_objects_and_maps.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\09_functions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\10_modules_and_packages.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\beginner\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\01_dom_basics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\02_events.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\03_array_methods.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\04_objects_and_prototypes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\05_error_handling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\06_async_javascript_and_promises.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\07_fetch_and_apis.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\08_npm_scripts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\09_testing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\10_project_organization.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\01_advanced_regex.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\02_context.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\03_typeglobs_and_symbols_overview.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\04_moose_and_moo.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\05_performance.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\06_security.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\07_async_and_processes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\08_distribution.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\09_legacy_code.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\10_architecture.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\01_setup_and_script_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\02_scalars.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\03_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\04_conditionals.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\05_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\06_arrays.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\07_hashes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\08_subroutines.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\09_regular_expressions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\10_modules.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\beginner\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\01_references.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\02_files.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\03_regex_in_practice.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\04_cpan.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\05_error_handling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\06_packages.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\07_objects.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\08_testing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\09_cli_scripts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\10_project_layout.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\01_dependency_injection.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\02_routing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\03_security.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\04_authentication.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\05_apis.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\06_performance.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\07_queues_and_jobs.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\08_deployment.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\09_framework_overview.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\10_architecture.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\01_setup_and_script_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\02_variables_and_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\03_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\04_conditionals.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\05_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\06_arrays.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\07_associative_arrays.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\08_functions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\09_includes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\10_composer_basics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\beginner\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\01_forms.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\02_sessions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\03_files.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\04_errors_and_exceptions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\05_object_oriented_php.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\06_namespaces.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\07_composer.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\08_databases_with_pdo.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\09_testing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\10_small_web_apps.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\bash\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\bash\advanced\12_structured_data_interchange.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\bash\advanced\13_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\c\advanced\13_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\csharp\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\java\advanced\13_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\javascript\advanced\13_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\perl\advanced\13_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\php\advanced\13_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\11_defensive_input_validation.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\12_serialization_and_structured_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\06_sql_injection_and_parameterized_queries.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\01_iterators_and_generators.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\02_decorators.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\03_context_managers.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\04_async_python.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\05_performance_profiling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\06_packaging_and_publishing.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\07_application_configuration.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\08_logging_and_observability.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\09_architecture_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\10_deployment_basics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\advanced\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [x] `languages\python\beginner\01_setup_and_install.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `languages\python\beginner\02_project_and_file_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\03_variables_and_data_types.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\04_input_and_output.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\05_control_flow.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\06_loops.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\07_lists_and_collections.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\08_dictionaries_and_objects.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\09_functions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\beginner\10_modules_and_packages.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [x] `languages\python\beginner\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `languages\python\intermediate\01_virtual_environments_and_dependency_files.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\02_reading_and_writing_files.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\03_exceptions_and_error_handling.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\04_classes_and_objects.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\05_type_hints.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\06_working_with_json_and_apis.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\07_testing_with_pytest.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\08_project_organization.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\09_useful_standard_library_modules.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\10_packaging_small_tools.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\intermediate\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `languages\python\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\algorithms\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\algorithms\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\algorithms\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\algorithms\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\algorithms\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\algorithms\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\data-structures\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\data-structures\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\data-structures\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\data-structures\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\data-structures\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\data-structures\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\debugging\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\debugging\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\debugging\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\debugging\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\debugging\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\debugging\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\docker-basics\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\docker-basics\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\docker-basics\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\docker-basics\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\docker-basics\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\docker-basics\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [x] `topics\environment-variables\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `topics\environment-variables\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\environment-variables\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\environment-variables\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\environment-variables\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [x] `topics\environment-variables\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [x] `topics\git\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `topics\git\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\git\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\git\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\git\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [x] `topics\git\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `topics\github\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\github\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\github\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\github\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\github\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\github\06_github_actions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\github\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\gitlab\01_projects_and_merge_requests.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\gitlab\02_gitlab_ci_cd_pipelines.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\gitlab\03_gitlab_runners.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\gitlab\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\gitlab\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\gitlab\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\http-and-apis\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-integration | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\http-and-apis\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\http-and-apis\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\http-and-apis\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\http-and-apis\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\http-and-apis\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-integration | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-integration | owner: audit-agent-dispatched | status: audit-dispatched | sources: pending
- [ ] `topics\json\02_json_structure.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\03_jsonpath_basics.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\04_querying_nested_data.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\05_querying_arrays.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\06_filters_and_conditions.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\07_querying_api_responses.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\08_querying_json_in_languages.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\09_common_query_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\10_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\json\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-integration | owner: audit-agent-dispatched | status: audit-dispatched | sources: pending
- [ ] `topics\markdown\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\markdown\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\markdown\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\markdown\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\markdown\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\markdown\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\oop\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\oop\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\oop\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\oop\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\oop\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\oop\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\package-managers\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-integration | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\package-managers\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\package-managers\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\package-managers\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\package-managers\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\package-managers\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-integration | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\regex\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\regex\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\regex\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\regex\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\regex\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\regex\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\security-basics\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\security-basics\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\security-basics\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\security-basics\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\security-basics\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\security-basics\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\sql\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [x] `topics\terminal\01_foundations.md` | type: lesson | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `topics\terminal\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\terminal\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\terminal\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\terminal\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [x] `topics\terminal\README.md` | type: readme | priority: core | path: automation-first | cluster: automation-foundations | owner: coordinator-validated | status: complete | sources: official-docs-reviewed
- [ ] `topics\testing\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\testing\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\testing\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\testing\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\testing\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\testing\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\web-basics\01_foundations.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\web-basics\02_core_concepts.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\web-basics\03_practical_patterns.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\web-basics\04_common_mistakes.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\web-basics\05_practice_project.md` | type: lesson | owner: unassigned | status: needs-audit | sources: pending
- [ ] `topics\web-basics\README.md` | type: readme | owner: unassigned | status: needs-audit | sources: pending
