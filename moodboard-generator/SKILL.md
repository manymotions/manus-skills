---
name: moodboard-generator
description: Turn a creative brief for a brand, interior, fashion line, event or project into five distinct visual directions, each rendered by two image models, as square images from your ManyMotions account. Trigger on intents like "make a moodboard for…", "mood board for my brand", "visual directions for…", "interior design moodboard", "做一个情绪板", "生成品牌视觉情绪板", "帮我做室内设计灵感板". Not for editing uploaded or reference images, brand logos, named artists or copyrighted characters, real people.
---

# AI Moodboard Generator

ManyMotions runs the request as a Workflow Studio pipeline in the user's account. In Manus the flow is text only: the tools return the request, the credits required, the progress and, at the end, the absolute URLs of the moodboard images.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/moodboard-generator/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"moodboard-generator": {"url": "https://manymotions.com/api/manus/moodboard-generator/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- Describe what the moodboard is for and the mood, colors, materials, era and style. Manus prepares the request and shows the credits required. When you confirm, ManyMotions writes five different visual directions (subject, setting, light, palette, style) and renders each with two image models.

## When NOT to use

- Editing uploaded or reference images, brand logos, named artists or copyrighted characters, real people. Say that this connector only makes new moodboard images from text and point to ManyMotions (https://manymotions.com) for the rest.

## Supported input

- New images from a text brief only: no uploaded or reference images. Original directions only: no brand logos, artworks or artists named as the style, copyrighted characters or real people.

## Tools (server `moodboard-generator`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_moodboard` | yes (connector) | none | Saves the request. `brief` (required): The creative brief: what the moodboard is for (brand, interior, fashion, event, project), the mood, colors, materials, era and style. Max 1500 characters.<br>Returns `project` (with `project.id`), `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_result` | yes | charged | Starts the run of `project_id`. Returns `state: "processing"` and `run.run_id`. |
| `get_result` | yes | none | Reads the run of `project_id` (the project id, not a run id): `state`, `message`, `run` (`completed_steps`, `total_steps`, `credits_charged`), `outputs` (each with `type` and absolute `url`) when completed, `dashboard_url`. Read-only. |

## Flow

1. Collect what `prepare_moodboard` needs; ask only for what is missing.
2. Call `prepare_moodboard`. Show what came back: the request, `credits_required` and, when present, the remaining `balance`.
3. Wait for an explicit go-ahead ("yes", "generate it", "go"). Never call `generate_result` on your own initiative: it charges the credits of the user's ManyMotions account. To change the request, call `prepare_moodboard` again with the new values and use the new `project.id`.
4. Call `generate_result` with `project_id`. Expected `state: "processing"` with `run.run_id`.
5. Poll `get_result` with `project_id` every 15-30 seconds and report progress as `run.completed_steps/run.total_steps`; do not quote a time beyond "a few minutes".
6. On `completed`, give the user every absolute URL in `outputs` (the images) and `dashboard_url` to open the run in ManyMotions.

## Sign-in

In Manus the ManyMotions account is linked when the connector is connected: Connect in the connector settings opens the ManyMotions sign-in (a new user can create the account there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that sign-in is required, ask the user to open the connector in Manus settings and press Connect, then retry the same call with the same `project_id`. No API key is ever needed.

## States returned by the tools and how to handle them

- `ready`: request saved, nothing generated yet (also returned by `get_result` before `generate_result`).
- `processing`: running; keep polling and report the steps.
- `completed`: done; deliver the URLs in `outputs`. If `outputs` is empty, say the run finished without a file and give `dashboard_url`.
- `failed`: relay `message` (it says what went wrong) and offer to retry with the same or an adjusted request.
- `stopped`: the run was stopped on the ManyMotions side; give `dashboard_url`.
- `unavailable`: generation is temporarily paused on ManyMotions; relay the `message` and suggest trying later.
- `billing_required`: the account has no verified paid access. Explain it in one sentence and give `dashboard_url`; do not suggest plans, upgrades or purchases.
- `insufficient_credits`: not enough credits (`credits_required` versus `balance`). Explain it and give `dashboard_url`; do not suggest purchases.
- Error "Resource not found" or "Project not found": prepare the request again.

## Rules

- One go-ahead per run; a new moodboard images needs a new `prepare_moodboard`.
- Voices and images are AI-generated; never describe them as real people.
- Quote `credits_required` and `run.credits_charged` from the tool results; never invent prices or credit amounts.
