---
name: short-film-maker
description: Turn a logline into a narrative AI short film with characters, dialogue and a film look across several scenes, in drama, sci-fi, thriller, romance, mystery or fantasy, delivered as an MP4 from your ManyMotions account. Trigger on intents like "make a short film about…", "a sci-fi short film where…", "turn this logline into a film", "a short drama with dialogue", "做一部微电影", "把这个故事梗概拍成短片", "生成一部科幻短片". Not for feature-length films, existing franchises or real people, editing uploaded footage, single cinematic shots.
---

# Short Film Maker: AI Short Films with Story & Dialogue

The ManyMotions Video Agent produces the video in seven steps: Creative direction, Script, Prompts, Keyframes, Motion, Video, Final cut. In Manus the flow is text only: the tools return the settings, the credits required, the progress and, at the end, the absolute URL of the MP4.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/short-film-maker/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"short-film-maker": {"url": "https://manymotions.com/api/manus/short-film-maker/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- Give Manus the logline, the genre, the main characters, the setting and how it should end, plus language, format and length. Manus prepares the brief and shows the scenes and the credits required. When you confirm, the ManyMotions Video Agent writes the script scene by scene with one turn and an ending that lands, generates the keyframes and the scenes with a deliberate film look and the dialogue, and edits the final cut.

## When NOT to use

- Feature-length films, existing franchises or real people, editing uploaded footage, single cinematic shots. Say that this connector only makes new videos from text and point to ManyMotions (https://manymotions.com) for the rest.
- New films from text only: no uploaded footage, no real actors. 250 credits per scene (8 s = 2 scenes, 15 s = 3, 30 s = 5, 45 s = 7, 60 s = 8).

## Tools (server `short-film-maker`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_short_film` | yes (connector) | none | Saves the brief. `logline` (required): The story in one or two sentences: who wants what, what stands in the way. Max 1500 characters.<br>`genre`: drama (default), sci-fi, thriller, romance, mystery or fantasy.<br>`characters`: Main characters: name, look, what they want. Max 800 characters.<br>`setting`: Where and when it happens. Max 300 characters.<br>`ending`: How it should end, optional. Max 300 characters.<br>`language`: Spoken language. Default English. Values: English, Spanish, Portuguese, Korean, Japanese, Mandarin, Indonesian.<br>`format`: Aspect ratio: 16:9 (default for a film look), 9:16, 1:1 or 4:5.<br>`duration`: Target length: 8s = 2 scenes, 15s = 3 (default), 30s = 5, 45s = 7, 60s = 8. 250 credits per scene.<br>Returns `project` (with `project.id` and the brief), `scenes`, `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_video` | yes | charged | Starts the production of `project_id` (250 credits per scene). Returns `state` and `run` (`run_id`, `completed_steps`, `total_steps`, `current_step`). |
| `get_video` | yes | none | Reads the run of `project_id` (the project id, not a run id): `state`, `run` (steps, `run.script` once written, `run.error`), `video_url` when ready, `dashboard_url`. Read-only. |

## Flow

1. Collect what the prepare tool needs; ask only for what is missing, the defaults cover the rest.
2. Call `prepare_short_film`. Show the plan: the main settings, `scenes`, `credits_required` and, when present, the remaining `balance`.
3. Wait for an explicit go-ahead ("yes", "produce it", "go"). Never call `generate_video` on your own initiative: it charges the credits of the user's ManyMotions account. To change the brief, call `prepare_short_film` again with the new values and use the new `project.id`.
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

- One go-ahead per production; a second video needs a new `prepare_short_film`.
- Presenters, characters and voices are AI-generated; never describe them as real people.
- Quote `scenes` and `credits_required` from the tool results; never invent prices, credit amounts or production times beyond "several minutes".
