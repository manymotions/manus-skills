---
name: church-event-video
description: Turn the details of a service, holiday celebration, conference, retreat or community gathering into a warm invitation video with a welcome spoken to camera, delivered as an MP4 from your ManyMotions account. Trigger on intents like "make an invitation video for our church…", "Christmas service invitation video", "youth retreat promo video", "community iftar welcome video", "做一个教会活动邀请视频", "生成一段圣诞礼拜邀请视频", "做个社区聚会邀请短片". Not for sermons or religious teaching, fundraising appeals, uploaded footage or photos, still flyers.
---

# Church & Faith Event Video Maker: Event Invitations

The ManyMotions Video Agent produces the video in seven steps: Creative direction, Script, Prompts, Keyframes, Motion, Video, Final cut. In Manus the flow is text only: the tools return the settings, the credits required, the progress and, at the end, the absolute URL of the MP4.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/church-event-video/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"church-event-video": {"url": "https://manymotions.com/api/manus/church-event-video/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- Tell Manus who is inviting (the church, parish, mosque, temple, synagogue or community), the kind of event (service, holiday celebration, conference, retreat, youth or community event), what happens, when and where, and the invitation in your own words, plus language, format and length. Manus prepares the brief and shows the scenes and the credits required. When you confirm, the ManyMotions Video Agent writes the invitation scene by scene (welcome, what the event is, what it feels like, an open invitation with no pressure), generates the scenes of the gathering and edits the final cut.

## When NOT to use

- Sermons or religious teaching, fundraising appeals, uploaded footage or photos, still flyers. Say that this connector only makes new videos from text and point to ManyMotions (https://manymotions.com) for the rest.
- New videos from text only: the people and places are generated from your description, photos of your community cannot be used. 250 credits per scene (8 s = 2 scenes, 15 s = 3, 30 s = 5, 45 s = 7, 60 s = 8).

## Tools (server `church-event-video`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_church_event_video` | yes (connector) | none | Saves the brief. `community` (required): Who is inviting: the church, parish, mosque, temple, synagogue or community and what it is like. Max 300 characters.<br>`event_type`: service (default), holiday, conference, retreat, youth-event or community-event.<br>`event_details` (required): What happens, when and where. Max 1000 characters.<br>`message`: The welcome or invitation in the community's own words. Max 500 characters.<br>`language`: Spoken language. Default English. Values: English, Spanish, Portuguese, Korean, Japanese, Mandarin, Indonesian.<br>`format`: Aspect ratio: 9:16 (default, TikTok, Reels, Shorts), 1:1, 4:5 or 16:9 (YouTube).<br>`duration`: Target length: 8s = 2 scenes, 15s = 3 (default), 30s = 5, 45s = 7, 60s = 8. 250 credits per scene.<br>Returns `project` (with `project.id` and the brief), `scenes`, `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_video` | yes | charged | Starts the production of `project_id` (250 credits per scene). Returns `state` and `run` (`run_id`, `completed_steps`, `total_steps`, `current_step`). |
| `get_video` | yes | none | Reads the run of `project_id` (the project id, not a run id): `state`, `run` (steps, `run.script` once written, `run.error`), `video_url` when ready, `dashboard_url`. Read-only. |

## Flow

1. Collect what the prepare tool needs; ask only for what is missing, the defaults cover the rest.
2. Call `prepare_church_event_video`. Show the plan: the main settings, `scenes`, `credits_required` and, when present, the remaining `balance`.
3. Wait for an explicit go-ahead ("yes", "produce it", "go"). Never call `generate_video` on your own initiative: it charges the credits of the user's ManyMotions account. To change the brief, call `prepare_church_event_video` again with the new values and use the new `project.id`.
4. Call `generate_video` with `project_id`. Expected `state: "processing"` with `run.run_id`.
5. The run takes several minutes. Poll `get_video` with `project_id` every 20-30 seconds and report progress as `completed_steps/total_steps` and `current_step`; once `run.script` is present you may show the script to the user.
6. On `completed`, give the user the absolute `video_url` of the MP4 and `dashboard_url` to open the video in ManyMotions.

## Sign-in

In Manus the ManyMotions account is linked when the connector is connected: Connect in the connector settings opens the ManyMotions sign-in (a new user can create the account there) and Manus keeps the connection. Every tool, the `prepare_` tool included, runs on that connection. If a call answers that sign-in is required, ask the user to open the connector in Manus settings and press Connect, then retry the same call with the same `project_id`. No API key is ever needed.

## States returned by the tools and how to handle them

- `ready`: brief saved, nothing produced yet (also returned by `get_video` when no run exists).
- `processing`: producing; keep polling and report the steps.
- `completed`: done; deliver `video_url`.
- `failed`: say which step failed, quoting `run.error` if present, and offer to retry with the same or an adjusted brief.
- `stopped`: the run was stopped on the ManyMotions side; give `dashboard_url`.
- `unavailable`: generation is temporarily paused on ManyMotions; relay the `message` and suggest trying later.
- `billing_required`: the account has no verified paid access. Explain it in one sentence and give `dashboard_url`; do not suggest plans, upgrades or purchases.
- `insufficient_credits`: not enough credits for this video (`credits_required` versus `balance`). Explain it and give `dashboard_url`; do not suggest purchases.
- `account_required`: the call ran without an account. Ask the user to press Connect on the connector in Manus settings and retry, or give `dashboard_url`, where the brief is already loaded.
- Error "Project not found": prepare the brief again. Error "not supported": use one of the values listed above.

## Rules

- One go-ahead per production; a second video needs a new `prepare_church_event_video`.
- Presenters, characters and voices are AI-generated; never describe them as real people.
- Quote `scenes` and `credits_required` from the tool results; never invent prices, credit amounts or production times beyond "several minutes".
