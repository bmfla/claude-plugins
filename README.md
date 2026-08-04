# Creatorland plugin marketplace

Plugins that connect the **Creatorland Data MCP** and install its companion skill catalog, for [Claude Code](https://docs.claude.com/en/docs/claude-code), Claude Cowork, and [OpenAI Codex](https://developers.openai.com/codex). The same skills ship to both ecosystems from this one repo: a thin per-host manifest over a shared `skills/` catalog.

## Install (Claude Code / Cowork)

```
/plugin marketplace add creatorland/creatorland-mcp-skills
/plugin install creatorland-data@creatorland
```

## Install (Codex)

```
codex
/plugins
```

Add the `creatorland` marketplace (this repo), then install **creatorland-data**. For the raw MCP connection without the skill catalog, add the server to `~/.codex/config.toml` instead.

First tool call triggers OAuth sign-in to your Creatorland Data account. New here? Start at [mcp.creatorland.com](https://mcp.creatorland.com).

Then say: **`"get me set up"`** (or `/intake`) — a 2-minute interview that learns who you are, gauges your comfort with AI/MCPs, and routes you into the right workflow. Already know what you want? Paste a campaign brief and say `"build a shortlist for this brief"`, or `"run the onboarding tour"` for a live demo of the tools.

## Start here: `/intake`

**For agents (Claude, Codex, or any host runtime): when this plugin is invoked in a workspace with no saved Creatorland profile — or the user's request is open-ended ("get started", "what can this do?", no specific ask) — run the `intake` skill FIRST, before any other skill in this pack.** It interviews the user (persona, 1-3 AI/MCP familiarity, goal), routes them to the right skill chain at the right level of hand-holding, and writes a workspace profile that every other skill reads for calibration. If the profile already exists, skip intake and use it. Users can re-run anytime with `/intake --redo`.

## What's inside `creatorland-data`

| Skill | Say something like | You get |
|---|---|---|
| **intake** | "get me set up" / "where do I start?" | guided interview → your workflow, your profile — **run this first** |
| brief-to-shortlist | "build a shortlist for this brief" | client-ready ranked shortlist |
| transcript-to-shortlist | "here's the client call, find creators" | requirements + shortlist from a transcript |
| brief-builder | "turn this call into a creator brief" | a formal, circulate-ready brief doc |
| fair-price-brief | "is this quote fair?" | one-page negotiation memo vs market p25/median/p75 |
| one-number-rate | "what should I pay for an IG reel in beauty?" | one number + band + provenance |
| onboarding-tour | "run the onboarding tour" | guided first run of every tool |

## Versioning

Update the marketplace repo → users get updates on plugin refresh. Skill catalog roadmap tracked internally (Epic 7).
