---
name: cinematic-video
description: Describe a scene, pick one of 61 named camera moves (dolly, crane, orbit, crash zoom, dolly zoom, FPV drone, bullet time) plus shot, lighting and look, and render a cinematic text-to-video shot with Veo, Kling, Seedance or Wan in your ManyMotions account. Trigger on intents like "make a cinematic shot of…", "dolly zoom video of…", "FPV drone shot through…", "crane up over…", "生成一个电影感镜头", "用推轨运镜拍一段…", "做一段穿越机航拍视频". Not for image-to-video, editing or upscaling uploaded footage, multi-scene stories with dialogue.
---

# Cinematic AI Video Maker with Camera Moves

ManyMotions renders the shot with the cinema video model you pick. In Manus the flow is text only: the tools return the settings, the credits required, the progress and, at the end, the absolute URL of the video.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/cinematic-video/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"cinematic-video": {"url": "https://manymotions.com/api/manus/cinematic-video/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- Describe the scene and pick one camera move from the Cinema Studio catalog of 61 moves (dolly in and out, crane up, 360 orbit, crash zoom, dolly zoom, FPV drone, bullet time, whip pan, hyperlapse and more), plus shot size, lighting, look, format, length and a cinema video model (Veo 3.1, Kling 3.0, Seedance 2.0, Wan 2.7). Manus prepares the shot and shows the settings and the credits required. When you confirm, ManyMotions renders the clip and Manus returns the video link.

## When NOT to use

- Image-to-video, editing or upscaling uploaded footage, multi-scene stories with dialogue. Say that this connector only makes new text-to-video shots from text and point to ManyMotions (https://manymotions.com) for the rest.
- Text-to-video only: no image-to-video and no editing of uploaded footage. Credits depend on the model and the length and are shown before rendering.

## Tools (server `cinematic-video`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_cinematic_video` | yes (connector) | none | Saves the shot. `scene` (required): What happens in the shot: subject, action and setting. Max 1500 characters.<br>`camera_move`: one of 61 move ids, e.g. dolly-in, dolly-zoom-in, crane-up, 360-orbit, crash-zoom-in, fpv-drone, bullet-time, whip-pan, hyperlapse, static; the tool schema lists every id with its name.<br>`shot`: Shot size or angle the clip opens on. Values: extreme-wide, wide, medium, medium-close-up, close-up, extreme-close-up, low-angle, high-angle, over-the-shoulder, aerial.<br>`lighting`: Lighting, e.g. golden hour backlight, neon noir, candlelight, overcast daylight. Max 200 characters.<br>`look`: Color grade or film look, e.g. teal and orange, 35mm film grain, black and white, anamorphic flares. Max 200 characters.<br>`format`: landscape 16:9 (default), vertical 9:16 or square 1:1 (square not on Veo).<br>`duration`: Length in seconds; must be one the model supports (Kling 5/10/15, Veo 4/6/8, Seedance 4-15, Wan 3-15). Default: the model's shortest.<br>`model_id`: cinema video model: Veo 3.1, Veo 3.1 Fast, Kling 3.0 Pro, Kling 3.0 Standard, Kling 2.6 Pro, Seedance 2.5, Wan 2.7, Seedance 2.0, Seedance 2.0 Fast, Kling O3 Pro, Seedance v1.5 Pro (ids in the tool schema). Default Kling 3.0 Pro.<br>Returns `project` (with `project.id`), the resolved settings, `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_media_forge` | yes | charged | Renders the prepared `project_id`. Returns `state` and `job` (`job.id`). |
| `get_media_forge_job` | yes | none | Reads the job `job_id`: `state`, `job.result.video` when ready, `job.error`, `dashboard_url`. It never starts a new render. |

## Flow

1. Collect the scene and the camera move, plus shot, lighting, look, format, length and model if the user cares; the defaults cover the rest.
2. Call `prepare_cinematic_video`. Show what came back: model name, camera move, format, duration and `credits_required`; if `balance` is present, show the remaining credits too.
3. Wait for an explicit confirmation ("yes", "render it", "go"). Never call `generate_media_forge` on your own initiative: it spends the credits of the user's ManyMotions account. To change the shot, call `prepare_cinematic_video` again and use the new `project.id`.
4. Call `generate_media_forge` with `project_id`. Expected `state: "processing"` (or `queued`) with `job.id`.
5. Poll `get_media_forge_job` with `job_id` at short intervals until `state` is `completed` or `failed`; do not quote a rendering time to the user.
6. On `completed`, give the user the absolute URL in `job.result.video`, plus `dashboard_url` to open the project in ManyMotions.

## Sign-in

In Manus the ManyMotions account is linked when the connector is connected: Connect in the connector settings opens the ManyMotions sign-in (a new user can create the account there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that sign-in is required, ask the user to open the connector in Manus settings and press Connect, then retry the same call with the same `project_id`. No API key is ever needed.

## States returned by the tools and how to handle them

- `ready`: shot saved, nothing rendered yet.
- `processing` / `queued`: rendering; keep polling.
- `completed`: done; deliver the video URL.
- `failed`: say that the render failed and quote `job.error` if present; the server refunds the credits of a failed job. Offer to retry or to change the scene or the model.
- `billing_required`: the account has no verified paid access. Explain it in one sentence and give `dashboard_url`; do not suggest plans, upgrades or purchases.
- `insufficient_credits`: not enough credits for this render (`credits_required` versus `balance`). Explain it and give `dashboard_url`; do not suggest purchases.
- `account_required`: the call ran without an account. Ask the user to press Connect on the connector in Manus settings and retry, or give `dashboard_url`, where the shot is already loaded.
- `pending_payment` on a job: the render is parked until the account can pay; give `dashboard_url`.
- Error "Project not found" or "Job not found": prepare the shot again. Error "not supported": use one of the values listed above.

## Rules

- One confirmation per render. A new shot means a new `prepare_cinematic_video`.
- Quote `credits_required` from the tool result; never invent prices, credit amounts or rendering times.
- Report model names and settings exactly as returned by the tools.
