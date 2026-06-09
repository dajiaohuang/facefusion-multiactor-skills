# Facefusion MultiActor Skills

Phase-based skills for coordinating a multi-actor FaceFusion workflow across Codex and other agent runtimes.

This repo packages the conversation layer behind the multi-actor flow, separate from the main [FaceFusion MCP](https://github.com/dajiaohuang/facefusion-mcp) execution plugin. It is meant for cases where you want to reuse the agent workflow itself:

- create the first draft `cast.json` and `shots.json`
- discover target-face clusters from the target video
- merge clusters into final roles and bind source faces
- add optional shot-level operations such as `lip_sync` or `face_enhance`
- review the full configuration before execution
- move through preview approval and targeted retry

This repository is best paired with a real FaceFusion MCP tool surface. The recommended execution backend is [FaceFusion MCP](https://github.com/dajiaohuang/facefusion-mcp).

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

In other words:

- this repo decides how an agent should talk through the multi-actor workflow
- the MCP/plugin repo does the actual FaceFusion work

## Flow

1. Draft cast and shots.
2. Run reference discovery.
3. Merge detected clusters and bind source faces.
4. Add optional shot-level operations if needed.
5. Build the plan.
6. Return the full configuration for explicit confirmation.
7. Only then materialize preview or final jobs.

## Install In Codex

This is the most natural runtime for this repository.

1. Install [FaceFusion MCP](https://github.com/dajiaohuang/facefusion-mcp) so the `facefusion_*` tools actually exist.
2. Copy or install this repo's `skills/` and `resources/` into your Codex skills area.
3. Keep the relative layout intact so the skill references still resolve:

```text
resources/
skills/
```

4. Reload Codex skills.
5. Start with the `coordinate-multi-actor-facefusion` skill.

## Install In OpenClaw

OpenClaw can use this repo as an instruction pack on top of the FaceFusion MCP server.

1. Install and configure [FaceFusion MCP](https://github.com/dajiaohuang/facefusion-mcp) in the same OpenClaw environment.
2. Copy this repo into a project knowledge or skill directory that OpenClaw can read.
3. Expose these files to the agent:
   - `skills/coordinate-multi-actor-facefusion/SKILL.md`
   - `skills/draft-facefusion-cast-and-shots/SKILL.md`
   - `skills/review-facefusion-references/SKILL.md`
   - `skills/refine-facefusion-plan/SKILL.md`
   - `resources/multi-actor-workflow.md`
   - `resources/parameter-mapping.md`
4. Tell OpenClaw to use these files as workflow rules, not as executable tools.
5. Let OpenClaw call the `facefusion_*` MCP tools from the FaceFusion MCP repo.

OpenClaw is a good fit when source images may arrive remotely later and the workflow needs repeated confirmation turns.

## Install In Hermes

Hermes should treat this repo as reusable workflow instructions layered over the FaceFusion MCP backend.

1. Install [FaceFusion MCP](https://github.com/dajiaohuang/facefusion-mcp) first.
2. Mount this repo where Hermes can load project instructions or local knowledge files.
3. Feed the four `SKILL.md` files plus the two `resources/*.md` files into the Hermes prompt or project context.
4. Keep the execution boundary clear:
   - Hermes reads this repo for process
   - Hermes calls FaceFusion MCP for execution
5. Start from the top-level `coordinate-multi-actor-facefusion` flow.

## Install In Claude

Claude can use the workflow, but not as native Codex skills.

1. Install [FaceFusion MCP](https://github.com/dajiaohuang/facefusion-mcp) in the Claude runtime first.
2. Copy the relevant files from this repo into Claude project knowledge, shared context, or project instructions.
3. Treat the files as structured operating instructions:
   - phase routing
   - role/reference review
   - confirmation gates
   - preview approval and retry rules
4. Ask Claude to follow the top-level workflow in `skills/coordinate-multi-actor-facefusion/SKILL.md`.
5. Use the FaceFusion MCP tools for all actual execution.

Claude can use this repo effectively, but `SKILL.md` is documentation for Claude, not a native feature format.

## Recommended Pairing

Use both repos together when you want the full experience:

1. `facefusion-mcp`
   - execution tools
   - install/setup automation
   - queueing
   - UI launch
   - plan rendering
2. `facefusion-multiactor-skills`
   - phase-by-phase conversation flow
   - merge/source decision logic
   - confirmation discipline
   - preview/retry orchestration rules

## Note

This repo contains the skill layer only, not the full FaceFusion MCP server/plugin bundle.
