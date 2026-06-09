# coordinate-multi-actor-facefusion-skills

Phase-based Codex skills for coordinating a multi-actor FaceFusion workflow.

This repo packages the conversation-layer skills behind the multi-actor flow, separate from the larger FaceFusion MCP plugin. It is meant for cases where you want to reuse the agent workflow itself:

- create the first draft `cast.json` and `shots.json`
- discover target-face clusters from the target video
- merge clusters into final roles and bind source faces
- add optional shot-level operations such as `lip_sync` or `face_enhance`
- review the full configuration before execution
- move through preview approval and targeted retry

## Included Skills

- `skills/coordinate-multi-actor-facefusion`
- `skills/draft-facefusion-cast-and-shots`
- `skills/review-facefusion-references`
- `skills/refine-facefusion-plan`

## Included References

- `resources/multi-actor-workflow.md`
- `resources/parameter-mapping.md`

These files are kept so the copied skills still resolve their relative references.

## Structure

```text
resources/
skills/
  coordinate-multi-actor-facefusion/
  draft-facefusion-cast-and-shots/
  review-facefusion-references/
  refine-facefusion-plan/
```

## Intended Use

These skills are designed to sit on top of a FaceFusion tool surface. They do not execute FaceFusion by themselves. The expected runtime already has tool access for actions such as:

- `facefusion_define_cast`
- `facefusion_plan_shots`
- `facefusion_discover_role_references`
- `facefusion_apply_reference_decisions`
- `facefusion_apply_shot_operation_decisions`
- `facefusion_build_multi_actor_plan`
- `facefusion_review_multi_actor_configuration`
- `facefusion_materialize_multi_actor_jobs`
- `facefusion_approve_preview`
- `facefusion_retry_failed_task`

## Flow

1. Draft cast and shots.
2. Run reference discovery.
3. Merge detected clusters and bind source faces.
4. Add optional shot-level operations if needed.
5. Build the plan.
6. Return the full configuration for explicit confirmation.
7. Only then materialize preview or final jobs.

## Note

This repo contains the skill layer only, not the full FaceFusion MCP server/plugin bundle.
