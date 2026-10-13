---
name: ai-logo-maker
description: Design logo concepts from a brand brief with ManyMotions Logo Maker — collect brand name, industry, style and colors, render up to four PNG concepts and return their links. Trigger on intents like "design a logo for…", "logo ideas for my brand", "make me a brand mark", "设计一个logo", "帮我做个品牌标志", "生成几个logo方案". Not for editing an existing logo, SVG or vector export, or trademark checks.
---

# AI Logo Maker & Brand Designer

ManyMotions Logo Maker renders logo concepts from a written brief with an AI image model. In Manus the flow is text only: the tools return the settings, the credits required, the job state and, at the end, the absolute URLs of the PNG files.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/logomaker/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"ai-logo-maker": {"url": "https://manymotions.com/api/manus/logomaker/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- The user wants a new logo, brand mark or several logo concepts for a brand, product, shop, app, team or event.
- The user wants to change the model, aspect ratio, resolution or number of concepts of a logo project you already prepared.

## When NOT to use

- Editing or recreating an uploaded, attached or reference logo or image; background removal; SVG or vector export; video. Say that Logo Maker only creates new logo concepts from a text brief and that upload workflows live in ManyMotions Media Forge (https://manymotions.com).
- Trademark searches or legal clearance: never claim a name or mark is available.

## Tools (server `ai-logo-maker`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_logo_project` | yes (connector) | none | Saves a logo project from `prompt` (the brief, max 3000 characters); optional `model_id`, `model_name`, `ratio`, `resolution`, `count` (concepts, 1-4, default 4), `advanced_params`. Returns `project` (with `project.id` and the applied settings), `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_logo` | yes | charged | Renders the concepts of `project_id` in one job. Accepts the same settings as prepare for last-minute changes. Returns `state` and `job` (with `job.id`). |
| `get_logo_job` | yes | none | Reads the job `job_id`: `state`, `job.result.images` (PNG URLs), `job.error`, `dashboard_url`. Read-only. |

## Flow

1. Collect in conversation what the brief needs: brand name, industry, style (minimalist, vintage, playful, luxury...), colors, optional typography and mood. Ask only for what is missing, one short question at a time.
2. Write the prompt as a logo brief, for example `Logo design for <brand>, <industry>: <style>, <colors>, flat vector logo on a plain background, no mockup`, and call `prepare_logo_project`. "Flat vector" is a style instruction for the model: the output is still a raster PNG.
3. Show the user what came back: model name, aspect ratio, resolution, number of concepts and `credits_required`. If `balance` is present, show the remaining credits too.
4. Wait for an explicit confirmation ("yes", "generate", "go"). Never call `generate_logo` on your own initiative: it spends the credits of the user's ManyMotions account. If the user changes a setting (for example two concepts instead of four), pass it as an argument of `generate_logo` together with `project_id`.
5. Call `generate_logo` with `project_id`. Expected `state: "processing"` (or `queued`) with `job.id`.
6. Poll `get_logo_job` with `job_id` every few seconds until `state` is `completed` or `failed`.
7. On `completed`, list the absolute URLs from `job.result.images`, one per concept, plus `dashboard_url` to keep iterating in ManyMotions (variations, edits, exports happen there). Never illustrate the result with web or image-search pictures.

## Sign-in

In Manus the ManyMotions account is linked when the connector is connected: Connect in the connector settings opens the ManyMotions sign-in (a new user can create the account there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that sign-in is required, ask the user to open the connector in Manus settings and press Connect, then retry the same call with the same `project_id`. No API key is ever needed.

## States returned by the tools and how to handle them

- `ready`: project saved, nothing rendered yet.
- `processing` / `queued`: rendering; keep polling.
- `completed`: done; deliver the URLs.
- `failed`: say that the render failed and quote `job.error` if present; the server refunds the credits of a failed job. Offer to retry or to adjust the brief.
- `billing_required`: the account has no verified paid access. Explain it in one sentence and give `dashboard_url`; do not suggest plans, upgrades or purchases.
- `insufficient_credits`: not enough credits for this render. Explain it and give `dashboard_url`; do not suggest purchases.
- `account_required`: the call ran without an account. Ask the user to press Connect on the connector in Manus settings and retry, or give `dashboard_url`, where the brief and settings are already loaded.
- `pending_payment` on a job: the render is parked until the account can pay; give `dashboard_url`.
- Error "Project not found" or "Job not found": prepare the project again.

## Rules

- Outputs are raster PNG images. Never promise SVG, vector files, transparent backgrounds you have not seen, or trademark clearance.
- Quote `credits_required` from the tool result; never invent prices, credit amounts or rendering times.
- One confirmation per render; a new brief means a new `prepare_logo_project`.
