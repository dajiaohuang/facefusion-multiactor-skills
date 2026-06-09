---
name: refine-facefusion-plan
description: Refine plan details for a multi-actor FaceFusion project after role binding. Use when you need to add optional per-shot pipeline operations, build plan.json, review or summarize the draft plan, approve previews, or retry only the affected task instead of rebuilding the project.
---

# Refine Facefusion Plan

Use this skill once cast, shots, and reference decisions are already in place.

## Workflow

1. Read `../../resources/multi-actor-workflow.md`.
2. Ask whether any shot needs optional pipeline operations such as `lip_sync`, `face_enhance`, `frame_enhance`, `background_remove`, `expression_restore`, `face_edit`, `age_modify`, or `frame_colorize`. Default is off.
3. Apply those optional operations with `facefusion_apply_shot_operation_decisions` when needed.
4. Build the preview/final draft with `facefusion_build_multi_actor_plan`.
5. Before any swap execution, call `facefusion_review_multi_actor_configuration` and show the user the full current configuration for confirmation:
   - merged roles and source paths
   - shot ranges and role assignments
   - optional shot operations
   - preview mode and quality profile
   - whether anything is still unresolved
6. In UI mode, render `manifests/plan-view.html` with `facefusion_render_plan_ui`.
7. In no-UI mode, summarize the same confirmation payload in text and wait for explicit approval.
8. Only after explicit user confirmation should you materialize jobs with `facefusion_materialize_multi_actor_jobs`.
9. Use `facefusion_approve_preview` for preview promotion.
10. Use `facefusion_retry_failed_task` for targeted recovery.

## Required Tools

Plan and confirmation:

- `facefusion_apply_shot_operation_decisions`
- `facefusion_build_multi_actor_plan`
- `facefusion_review_multi_actor_configuration`
- `facefusion_render_plan_ui`

Execution and queue control:

- `facefusion_materialize_multi_actor_jobs`
- `facefusion_create_job`
- `facefusion_update_job_steps`
- `facefusion_run_jobs`
- `facefusion_manage_jobs`
- `facefusion_check_queue`

Approval and recovery:

- `facefusion_approve_preview`
- `facefusion_retry_failed_task`

If any execution tool is missing, do not silently skip to direct CLI assumptions. Surface the missing tool dependency explicitly and stop at the confirmation stage.

## Tool Invocation Map

Use this concrete tool sequence for plan refinement and execution lifecycle.

Optional shot operations:

1. Ask whether any shot needs extra operations such as `lip_sync`, `face_enhance`, `frame_enhance`, `background_remove`, `expression_restore`, `face_edit`, `age_modify`, or `frame_colorize`.
2. If yes, call:
   - `facefusion_apply_shot_operation_decisions(project_id=..., shot_operations=[...])`

Plan build:

1. Call:
   - `facefusion_build_multi_actor_plan(project_id=..., preview_mode=..., quality_profile=...)`
2. Then call:
   - `facefusion_review_multi_actor_configuration(project_id=...)`
3. If UI review helps:
   - `facefusion_render_plan_ui(project_id=...)`

Hard confirmation gate:

- After `facefusion_review_multi_actor_configuration(project_id=...)`, stop and return the full configuration to the user.
- Do not continue until the user explicitly confirms with language such as:
  - `confirm`
  - `run preview`
  - `proceed`

Preview execution:

1. After confirmation, call:
   - `facefusion_materialize_multi_actor_jobs(project_id=..., include_final_tasks=False)`
2. Submit and run the preview job with:
   - `facefusion_run_jobs(mode="submit", job_id=project_id)`
   - `facefusion_run_jobs(mode="run", job_id=project_id)`
3. If needed, inspect queue state with:
   - `facefusion_check_queue()`
   - `facefusion_manage_jobs(action="list")`

Preview promotion:

1. If the preview is approved, call:
   - `facefusion_approve_preview(project_id=..., task_id=... or shot_id=..., approved=True, approval_notes=...)`
2. If the preview is rejected, call:
   - `facefusion_approve_preview(project_id=..., task_id=... or shot_id=..., approved=False, approval_notes=...)`

Retry and recovery:

- For targeted recovery, call:
  - `facefusion_retry_failed_task(project_id=..., task_id=..., materialize_immediately=...)`
- For queue inspection or cleanup, use:
  - `facefusion_manage_jobs(action="list" | "delete" | "delete_all", ...)`

Do not skip the explicit confirmation gate between:

- `facefusion_build_multi_actor_plan(...)`
- and `facefusion_materialize_multi_actor_jobs(...)`

## Conversation Rules

- Optional shot pipeline additions are off by default.
- Prefer adding shot operations explicitly instead of implying them from vague quality language.
- Do not rebuild the entire project when only one preview or final task needs a retry.
- Never start swap execution immediately after role/source binding. Always insert one explicit full-configuration confirmation step first.

## Tool Rules

- Use `facefusion_apply_shot_operation_decisions` before plan build when the user asks for shot-level additions.
- Use `facefusion_review_multi_actor_configuration` after plan build and before any materialization or execution.
- Use `facefusion_render_plan_ui` when a visual review is helpful.
- Use `facefusion_approve_preview` instead of editing `plan.json` by hand.
- Use `facefusion_retry_failed_task` for localized failure recovery.

## References

- `../../resources/multi-actor-workflow.md`
- `references/plan-refinement-patterns.md`
