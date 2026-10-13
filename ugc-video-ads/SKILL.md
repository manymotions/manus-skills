---
name: ai-ugc-video-ads-maker
description: Produce a short UGC-style video ad with an AI presenter and voice from a product brief with the ManyMotions Marketing Agent, and follow the production until the MP4 is ready. Trigger on intents like "make a TikTok ad for my product", "create a UGC video ad", "testimonial video for my app", "product demo ad", "做一条带货视频广告", "生成一个UGC口播广告", "帮我做个产品短视频广告". Not for editing uploaded footage, captions on existing videos, real human creators, still images or logos.
---

# AI UGC Video Ads Maker

The ManyMotions Marketing Agent produces short video ads with an AI presenter and an AI voice in seven steps: Strategy, Script, Prompts, Keyframes, Motion, Video, Final cut. In Manus the flow is text only: the tools return the brief, the scene count, the credits required, the run progress and, at the end, the absolute URL of the MP4.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/ugc/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"ai-ugc-video-ads-maker": {"url": "https://manymotions.com/api/manus/ugc/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- The user wants a new short video ad for a product, brand, app or service: UGC talking head, testimonial, product demo, unboxing, tutorial, review, transformation, podcast-style clip.
- The user wants to adjust language, tone, format, length, template or speech speed of an ad brief you already prepared and not yet produced.

## When NOT to use

- Editing uploaded footage, adding captions to an existing video, hiring or simulating a specific real creator, still images, logos, videos that are not ads. Say that this connector only produces new AI video ads and point to ManyMotions (https://manymotions.com) for the rest.

## Tools (server `ai-ugc-video-ads-maker`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_ugc_video_ad` | yes (connector) | none | Saves the brief. Required: `product_description` (max 3000 characters), `target_audience` (max 300). Optional: `product_url`, `language` (English, Spanish, Portuguese, Korean, Japanese, Mandarin, Indonesian; default English), `tone` (energetic, friendly, professional, urgent; default energetic), `video_format` (9:16, 1:1, 4:5, 16:9; default 9:16), `duration` (8s, 15s, 30s, 45s, 60s; default 15s), `template` (ugc_ad, podcast_clip, testimonial, product_demo, transformation, tutorial, product_review, unboxing; default ugc_ad), `speech_speed` (slow, medium, medium-fast, fast; default medium-fast). Returns `project` (with `project.id` and the brief), `scenes`, `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_ugc_video_ad` | yes | charged | Starts the production of `project_id`. Accepts the brief fields for last-minute changes (applied only before a run exists). Returns `state` and `run` (`run_id`, `completed_steps`, `total_steps`, `current_step`). |
| `get_ugc_video_ad` | yes | none | Reads the run of `project_id` (note: the project id, not a job id): `state`, `run` (steps, `run.script` once written, `run.error`), `video_url` when ready, `dashboard_url`. Read-only. |

Scene count follows the length: 8s = 2 scenes, 15s = 3, 30s = 5, 45s = 7, 60s = 8. The credits are returned by the tool as `credits_required`; quote that number.

## Flow

1. Collect what the product is (and its URL if the user has one), who the ad targets, and any preference on language, tone, format, length, template or speech speed. Ask only for what is missing; defaults cover the rest.
2. Call `prepare_ugc_video_ad`. Show the plan: template, language, tone, format, length, `scenes`, `credits_required` and, when present, the remaining `balance`.
3. Wait for an explicit go-ahead ("yes", "produce it", "go"). Never call `generate_ugc_video_ad` on your own initiative: it charges the credits of the user's ManyMotions account. If the user changes a brief field, pass it as an argument of `generate_ugc_video_ad` together with `project_id`.
4. Call `generate_ugc_video_ad` with `project_id`. Expected `state: "processing"` with `run.run_id`.
5. The run takes several minutes. Poll `get_ugc_video_ad` with `project_id` every 20-30 seconds and report progress as `completed_steps/total_steps` and `current_step`; once `run.script` is present you may show the scene dialogue to the user.
6. On `completed`, give the user the absolute `video_url` of the MP4 and `dashboard_url` to open the ad in ManyMotions.

## Sign-in

In Manus the ManyMotions account is linked when the connector is connected: Connect in the connector settings opens the ManyMotions sign-in (a new user can create the account there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that sign-in is required, ask the user to open the connector in Manus settings and press Connect, then retry the same call with the same `project_id`. No API key is ever needed.

## States returned by the tools and how to handle them

- `ready`: brief saved, nothing produced yet (also returned by `get_ugc_video_ad` when no run exists).
- `processing`: producing; keep polling and report the steps.
- `completed`: done; deliver `video_url`.
- `failed`: say which step failed, quoting `run.error` if present, and offer to retry with the same or an adjusted brief.
- `stopped`: the run was stopped on the ManyMotions side; give `dashboard_url`.
- `unavailable`: generation is temporarily paused on ManyMotions; relay the `message` and suggest trying later.
- `billing_required`: the account has no verified paid access. Explain it in one sentence and give `dashboard_url`; do not suggest plans, upgrades or purchases.
- `insufficient_credits`: not enough credits for this ad (`credits_required` versus `balance`). Explain it and give `dashboard_url`; do not suggest purchases.
- `account_required`: the call ran without an account. Ask the user to press Connect on the connector in Manus settings and retry, or give `dashboard_url`, where the brief is already loaded.
- Error "Project not found": prepare the brief again. Error "Unsupported value": use one of the allowed values listed above.

## Rules

- One go-ahead per production; a second run of the same product needs a new `prepare_ugc_video_ad`.
- The presenter and the voice are AI-generated; never describe them as real creators.
- Quote `scenes` and `credits_required` from the tool results; never invent prices, credit amounts or production times beyond "several minutes".
