---
name: review-facefusion-references
description: Review and refine detected target roles for a multi-actor FaceFusion project. Use when cast.json and shots.json already exist and you need to run reference discovery, merge target clusters, bind source faces, use the reference review UI, or continue the same work in no-UI dialogue mode.
---

# Review Facefusion References

Use this skill for the role-binding and source-binding phase after the initial draft exists.

## Workflow

1. Read `../../resources/multi-actor-workflow.md`.
2. Run `facefusion_discover_role_references`.
3. If `default_ui_mode=true`, render `manifests/reference-view.html` with `facefusion_render_reference_ui` early.
4. Ask which detected clusters should merge into one final role.
5. Ask whether each merged role should keep or change the prefilled source face.
6. If some roles are still missing source images, allow them to remain unresolved temporarily when the user plans to send them later.
7. Apply resolved decisions with `facefusion_apply_reference_decisions`.

## Required Tools

Environment and discovery setup:

- `facefusion_health_check`
- `facefusion_list_capabilities`

Reference-review workflow:

- `facefusion_discover_role_references`
- `facefusion_render_reference_ui`
- `facefusion_apply_reference_decisions`

Optional troubleshooting helpers when discovery fails:

- `facefusion_download_models`
- `facefusion_check_queue`

If reference discovery fails because FaceFusion is not ready, missing models, or missing providers, stop and repair that first instead of improvising role-merge questions without a valid `references.json`.

## Tool Invocation Map

Use this concrete tool sequence for reference review.

Preparation:

1. If readiness is uncertain:
   - `facefusion_health_check()`
2. If discovery-related capabilities or models are questionable:
   - `facefusion_list_capabilities(category="all")`
   - `facefusion_download_models(...)` when the runtime is missing assets

Discovery:

1. Call:
   - `facefusion_discover_role_references(project_id=..., sample_frames_per_shot=..., cluster_distance_threshold=..., source_hint_names=[...]?)`
2. If UI review is preferred:
   - `facefusion_render_reference_ui(project_id=...)`

Review and bind:

1. Ask which detected clusters should merge.
2. Ask which source face each merged role should use.
3. Apply decisions with:
   - `facefusion_apply_reference_decisions(project_id=..., groups=[...], overwrite_cast=...)`

If the user is still missing source images:

- allow unresolved state temporarily
- do not build the execution plan yet
- wait for the user to send the missing source image, then rerun only:
  - `facefusion_apply_reference_decisions(project_id=..., groups=[...])`

Do not call in this phase:

- `facefusion_build_multi_actor_plan(...)`
- `facefusion_materialize_multi_actor_jobs(...)`
- `facefusion_run_jobs(...)`

## Conversation Rules

- Do not force manual role labeling before discovery when the footage is ambiguous.
- In UI mode, use the review page but keep the conversation going.
- In no-UI mode, summarize the same state in text and continue asking merge/source questions.
- If the user is working through OpenClaw or another remote image path, treat delayed source-image arrival as a normal partial state.

## Tool Rules

- Use `facefusion_discover_role_references` before asking merge questions.
- Use `facefusion_render_reference_ui` for visual review when available or preferred.
- Use `facefusion_apply_reference_decisions` for resolved merge/source updates.
- Do not build the final plan from this skill.

## References

- `../../resources/multi-actor-workflow.md`
- `references/reference-review-patterns.md`
