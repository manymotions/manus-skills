---
name: pitch-deck-generator
description: Turn your startup's description, problem, market and business model into six slide visuals (problem, solution, market, product, business model, the ask) designed as one deck, as images from your ManyMotions account. Trigger on intents like "make pitch deck slides for my startup", "investor deck visuals for…", "six slides for our pitch", "做一份融资路演 PPT 配图", "生成创业项目的路演幻灯片", "帮我做投资人演示文稿的画面". Not for editing uploaded decks, logos or images, other companies' logos or trademarks, financial advice.
---

# Pitch Deck Visual Generator

ManyMotions runs the request as a Workflow Studio pipeline in the user's account. In Manus the flow is text only: the tools return the request, the credits required, the progress and, at the end, the absolute URLs of the pitch deck slides.

## Connect in Manus

1. In Manus open Settings, then Connectors, and choose Create, then "Add MCP by URL".
2. Server URL: `https://manymotions.com/api/manus/pitch-deck-generator/mcp`. Or choose "Import MCP by JSON" with:

```json
{"mcpServers": {"pitch-deck-generator": {"url": "https://manymotions.com/api/manus/pitch-deck-generator/mcp"}}}
```

3. Save (not "Save and Publish"), then press Connect on the connector page and sign in to ManyMotions; a new user can create the account there.
4. "Try it out" opens a new task with the connector on; in other tasks, turn it on from the connector icons in the chat input.

## When to use

- Describe what the startup does, the problem it solves, its target market and its business model. Manus prepares the request and shows the credits required. When you confirm, ManyMotions plans the deck and renders six slides: problem, solution, market, product, business model and the ask.

## When NOT to use

- Editing uploaded decks, logos or images, other companies' logos or trademarks, financial advice. Say that this connector only makes new pitch deck slides from text and point to ManyMotions (https://manymotions.com) for the rest.

## Supported input

- Slide visuals from your own startup description only: no uploaded decks, logos or images, and no other companies' logos or trademarks on the slides.

## Tools (server `pitch-deck-generator`)

| Tool | Sign-in | Credits | Purpose |
|---|---|---|---|
| `prepare_pitch_deck` | yes (connector) | none | Saves the request. `startup` (required): What the startup does and its product. Max 1000 characters.<br>`problem` (required): The problem it solves. Max 600 characters.<br>`market` (required): The target market and customers. Max 600 characters.<br>`business_model` (required): How it makes money. Max 600 characters.<br>Returns `project` (with `project.id`), `credits_required`, `dashboard_url`, `balance` when signed in, `state: "ready"`. |
| `generate_result` | yes | charged | Starts the run of `project_id`. Returns `state: "processing"` and `run.run_id`. |
| `get_result` | yes | none | Reads the run of `project_id` (the project id, not a run id): `state`, `message`, `run` (`completed_steps`, `total_steps`, `credits_charged`), `outputs` (each with `type` and absolute `url`) when completed, `dashboard_url`. Read-only. |

## Flow

1. Collect what `prepare_pitch_deck` needs; ask only for what is missing.
2. Call `prepare_pitch_deck`. Show what came back: the request, `credits_required` and, when present, the remaining `balance`.
3. Wait for an explicit go-ahead ("yes", "generate it", "go"). Never call `generate_result` on your own initiative: it charges the credits of the user's ManyMotions account. To change the request, call `prepare_pitch_deck` again with the new values and use the new `project.id`.
4. Call `generate_result` with `project_id`. Expected `state: "processing"` with `run.run_id`.
5. Poll `get_result` with `project_id` every 15-30 seconds and report progress as `run.completed_steps/run.total_steps`; do not quote a time beyond "a few minutes".
6. On `completed`, give the user every absolute URL in `outputs` (the slide images) and `dashboard_url` to open the run in ManyMotions.

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

- One go-ahead per run; a new pitch deck slides needs a new `prepare_pitch_deck`.
- Voices and images are AI-generated; never describe them as real people.
- Quote `credits_required` and `run.credits_charged` from the tool results; never invent prices or credit amounts.
