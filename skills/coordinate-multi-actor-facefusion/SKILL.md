---
name: coordinate-multi-actor-facefusion
description: Coordinate a conversational multi-actor FaceFusion project through the FaceFusion MCP plugin. Use when the user wants to swap multiple roles or multiple faces across one or more videos, map source faces to named roles, define shot ranges, approve previews before final renders, or retry only the affected shot instead of rerunning the entire project.
---

# Coordinate Multi-Actor Facefusion

Use this skill as the top-level router for multi-role FaceFusion projects. It should decide which phase-specific skill to use next, keep the whole project coherent, and prevent the conversation from jumping into the wrong phase.

## Workflow

1. Start by reading `../../resources/multi-actor-workflow.md`.
2. If environment readiness is unknown, call `facefusion_health_check`.
3. Decide which phase the project is currently in:
   - draft cast and shots
   - review references and bind roles
   - refine plan and preview lifecycle
4. Delegate the detailed workflow to the matching phase skill:
   - `../draft-facefusion-cast-and-shots/SKILL.md`
   - `../review-facefusion-references/SKILL.md`
   - `../refine-facefusion-plan/SKILL.md`
5. After each phase action, summarize the updated project state and either move to the next phase or stay in the current phase if details are still missing.
6. Before any preview or final swap execution, require one dedicated confirmation turn that shows the full planned configuration back to the user.

## Standalone Tool Contract

This skill is intended to be installable on its own, but it still expects a FaceFusion tool surface to exist in the runtime.

Minimum environment and debugging tools:

- `facefusion_health_check`
- `facefusion_install_or_setup`
- `facefusion_list_capabilities`
- `facefusion_list_presets`
- `facefusion_check_queue`
- `facefusion_download_models`

Multi-actor workflow tools:

- `facefusion_define_cast`
- `facefusion_plan_shots`
- `facefusion_discover_role_references`
- `facefusion_apply_reference_decisions`
- `facefusion_apply_shot_operation_decisions`
- `facefusion_render_reference_ui`
- `facefusion_build_multi_actor_plan`
- `facefusion_review_multi_actor_configuration`
- `facefusion_render_plan_ui`
- `facefusion_materialize_multi_actor_jobs`
- `facefusion_approve_preview`
- `facefusion_retry_failed_task`

Queue execution tools:

- `facefusion_create_job`
- `facefusion_update_job_steps`
- `facefusion_run_jobs`
- `facefusion_manage_jobs`

If the environment/debugging tools are missing, stop and resolve FaceFusion readiness before trying to continue the workflow phases.

## Tool Invocation Map

Use these tool calls directly as the execution skeleton for this skill.

Environment bootstrap:

1. `facefusion_health_check()`
2. If readiness is incomplete:
   - inspect `needs_install`, `needs_setup`, `missing_components`
   - if the user wants repair or install, call `facefusion_install_or_setup(...)`
3. If the user asks what is available on this machine:
   - `facefusion_list_capabilities(category="all")`
   - `facefusion_list_presets()`
4. If queued or background work may affect the project:
   - `facefusion_check_queue()`

Phase routing by project state:

- no valid `cast.json` or `shots.json` yet:
  - delegate to `../draft-facefusion-cast-and-shots/SKILL.md`
- valid draft exists but no reviewed `references.json` decisions yet:
  - delegate to `../review-facefusion-references/SKILL.md`
- cast, shots, and reference decisions exist:
  - delegate to `../refine-facefusion-plan/SKILL.md`

Status-summary tools:

- summarize current state from:
  - `facefusion_review_multi_actor_configuration(project_id=...)` when a plan already exists
  - otherwise summarize from `cast.json`, `shots.json`, and `references.json` via the phase tools that produced them

Execution guard:

- never call `facefusion_materialize_multi_actor_jobs(...)`
- never call `facefusion_run_jobs(...)`
- until the user has explicitly confirmed the full configuration returned by `facefusion_review_multi_actor_configuration(project_id=...)`

## Conversation Rules

- Keep the conversation in one phase at a time.
- Ask only the minimum blocking question for the current phase.
- Do not mix initial draft collection, reference review, and preview approval into one long turn unless the user already provided nearly everything.
- Treat `default_ui_mode` as a routing preference, not a hard requirement.
- Always produce an initial draft state before trying to collect every possible detail.
- After merge decisions and source bindings are known, do not jump straight into execution. First return the full settings for confirmation.

## Tool Rules

- Stay in the multi-actor tool family unless the user abandons the project workflow.
- Use this skill to choose the next phase skill, not to duplicate all phase-specific instructions inline.
- If the user asks for a high-level project status, summarize `cast.json`, `shots.json`, `references.json`, and `plan.json` state rather than diving into one tool call immediately.
- Before materializing or running jobs, prefer `facefusion_review_multi_actor_configuration` so the user can approve the full configuration explicitly.

## References

- `../../resources/multi-actor-workflow.md`
- `../../resources/parameter-mapping.md`
- `references/conversation-patterns.md`
