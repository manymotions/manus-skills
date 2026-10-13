---
name: anime-story-video
description: Turn a story idea into an animated short in anime, cel, chibi, cartoon, storybook or 3D style, with recurring characters, dialogue and narration, delivered as an MP4 from your ManyMotions account. Trigger on intents like "make an anime short about…", "a cartoon video where…", "animate a bedtime story", "chibi animation of…", "做一个动漫短片", "生成一段卡通动画故事", "把这个故事做成动画". Not for animating uploaded drawings or photos, existing characters or franchises, editing footage, still images.
---

# Anime Story Video Maker: AI Cartoon & Animated Shorts

The ManyMotions Video Agent produces the video in seven steps: Creative direction, Script, Prompts, Keyframes, Motion, Video, Final cut. In Manus the flow is text only: the tools return the settings, the credits required, the progress and, at the end, the absolute URL of the MP4.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/anime-story-video/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"anime-story-video": {"url": "https://manymotions.com/api/manus/anime-story-video/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- Describe the story, the main characters, the art style (soft anime, flat cel, chibi, cartoon, storybook or 3D animation) and the mood, plus language, format and length. Manus prepares the brief and shows the scenes and the credits required. When you confirm, the ManyMotions Video Agent writes the script scene by scene, generates keyframes and animated scenes in one style held across the whole short, adds the voices and edits the final cut.

## When NOT to use

- Animating uploaded drawings or photos, existing characters or franchises, editing footage, still images. Say that this connector only makes new videos from text and point to ManyMotions (https://manymotions.com) for the rest.
- New videos from text only: no uploaded footage or reference drawings. 250 credits per scene (8 s = 2 scenes, 15 s = 3, 30 s = 5, 45 s = 7, 60 s = 8).

## Tools (server `anime-story-video`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_anime_story` | yes (connector) | none | Saves the brief. `story` (required): The story: premise, what happens, how it ends. Max 2000 characters.<br>`characters`: Main characters: name, look, personality. Max 800 characters.<br>`art_style`: anime (default), flat-cel, chibi, cartoon, storybook or 3d-animation.<br>`mood`: Mood, e.g. heartwarming, epic, funny, bittersweet. Max 120 characters.<br>`language`: Spoken language. Default English. Values: English, Spanish, Portuguese, Korean, Japanese, Mandarin, Indonesian.<br>`format`: Aspect ratio: 9:16 (default, TikTok, Reels, Shorts), 1:1, 4:5 or 16:9 (YouTube).<br>`duration`: Target length: 8s = 2 scenes, 15s = 3 (default), 30s = 5, 45s = 7, 60s = 8. 250 credits per scene.<br>Returns `project` (with `project.id` and the brief), `scenes`, `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_video` | yes | charged | Starts the production of `project_id` (250 credits per scene). Returns `state` and `run` (`run_id`, `completed_steps`, `total_steps`, `current_step`). |
| `get_video` | yes | none | Reads the run of `project_id` (the project id, not a run id): `state`, `run` (steps, `run.script` once written, `run.error`), `video_url` when ready, `dashboard_url`. Read-only. |

## Flow

1. Collect what the prepare tool needs; ask only for what is missing, the defaults cover the rest.
2. Call `prepare_anime_story`. Show the plan: the main settings, `scenes`, `credits_required` and, when present, the remaining `balance`.
3. Wait for an explicit go-ahead ("yes", "produce it", "go"). Never call `generate_video` on your own initiative: it charges the credits of the user's ManyMotions account. To change the brief, call `prepare_anime_story` again with the new values and use the new `project.id`.
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

- One go-ahead per production; a second video needs a new `prepare_anime_story`.
- Presenters, characters and voices are AI-generated; never describe them as real people.
- Quote `scenes` and `credits_required` from the tool results; never invent prices, credit amounts or production times beyond "several minutes".
