---
name: clipwave-video-image-generator
description: Generate AI images and AI videos from a text prompt with ManyMotions (text to image, text to video) and follow the render until the media link is ready. Trigger on intents like "generate an image of…", "make a short video of…", "render a 9:16 clip", "create a product shot", "生成一张图", "画一张…", "生成一段视频", "做个产品图". Not for editing uploaded images, image-to-video or video-to-video.
---

# AI Video & Image Generator by Clipwave

ManyMotions renders images and short videos from text with AI models. In Manus the flow is text only: the tools return the settings, the credits required, the job state and, at the end, the absolute URL of the file.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"clipwave-video-image-generator": {"url": "https://manymotions.com/api/manus/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- The user wants a new image or a new video from a description: product shot, illustration, cinematic clip, social video, concept art.
- The user wants to change model, aspect ratio, resolution, duration or number of images of a project you already prepared.

## When NOT to use

- The request starts from an uploaded, attached or reference image or video: editing, background replacement, product-photo retouching, image-to-video, video-to-video, remixing, upscaling. Say that this connector only does text-to-image and text-to-video and that upload workflows live in ManyMotions Media Forge (https://manymotions.com).
- Logos (use the Logo Maker connector) or video ads with a presenter (use the UGC Video Ads connector).

## Tools (server `clipwave-video-image-generator`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_media_forge_project` | yes (connector) | none | Saves a project from `tab` (`image` or `video`) and `prompt` (max 3000 characters); optional `model_id`, `model_name`, `ratio`, `resolution`, `duration` (video, seconds: 4, 5, 6, 8, 10, 15, 20), `count` (images, 1-4), `advanced_params`. Returns `project` (with `project.id` and the settings actually applied), `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_media_forge` | yes | charged | Starts the render of `project_id`. Accepts the same settings as prepare to apply last-minute changes. Returns `state` and `job` (with `job.id`). |
| `get_media_forge_job` | yes | none | Reads the job `job_id`: `state`, `job.result` with the media URLs, `job.error`, `dashboard_url`. Read-only. |

## Flow

1. As soon as the user describes an image or a video, call `prepare_media_forge_project`. Infer `tab`, write a clear prompt, pass only the settings the user stated; the server fills in the defaults.
2. Show the user what came back: model name, aspect ratio, resolution, duration, number of images and `credits_required`. If `balance` is present, show the remaining credits too.
3. Wait for an explicit confirmation ("yes", "generate", "go"). Never call `generate_media_forge` on your own initiative: it spends the credits of the user's ManyMotions account. If the user changes a setting, pass it as an argument of `generate_media_forge` together with `project_id` (no need to prepare again).
4. Call `generate_media_forge` with `project_id`. Expected `state: "processing"` (or `queued`) with `job.id`.
5. Poll `get_media_forge_job` with `job_id` at short intervals until `state` is `completed` or `failed`; do not quote a rendering time to the user.
6. On `completed`, give the user the absolute URLs from `job.result.images` (one per image) or `job.result.video`, plus `dashboard_url` to open the project in ManyMotions. Never replace the result with web or image-search pictures.

## Sign-in

In Manus the ManyMotions account is linked when the connector is connected: Connect in the connector settings opens the ManyMotions sign-in (a new user can create the account there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that sign-in is required, ask the user to open the connector in Manus settings and press Connect, then retry the same call with the same `project_id`. No API key is ever needed.

## States returned by the tools and how to handle them

- `ready`: project saved, nothing rendered yet.
- `processing` / `queued`: rendering; keep polling.
- `completed`: done; deliver the URLs.
- `failed`: say that the render failed and quote `job.error` if present; the server refunds the credits of a failed job. Offer to retry or to change the prompt or the model.
- `billing_required`: the account has no verified paid access. Explain it in one sentence and give `dashboard_url`; do not suggest plans, upgrades or purchases.
- `insufficient_credits`: not enough credits for this render (`credits_required` versus `balance`). Explain it and give `dashboard_url`; do not suggest purchases.
- `account_required`: the call ran without an account. Ask the user to press Connect on the connector in Manus settings and retry, or give `dashboard_url`, where the prompt and settings are already loaded.
- `pending_payment` on a job: the render is parked until the account can pay; give `dashboard_url`.
- Error "Project not found" or "Job not found": prepare the project again.

## Rules

- One confirmation per render. A new request with different content means a new `prepare_media_forge_project`.
- Quote `credits_required` from the tool result; never invent prices, credit amounts or rendering times.
- Report model names and settings exactly as returned by the tools.
