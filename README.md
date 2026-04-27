# Drumfire

A personal coffee roast logging system for hobbyist roasters. Talk to Claude naturally — it reads and writes structured JSON files. No UI, no framework, no setup beyond opening a conversation.

**Current phase:** Claude prototype. The goal is to discover the right data model through real use, then build a web app on top of it.

## How It Works

Drumfire has three modes, invoked as slash commands in Claude Code:

- `/plan` — pre-roast planning: reviews bean history, surfaces the last adjustment, sets a hypothesis
- `/roast` — active roasting: real-time logging, phase guidance, live diagnostics
- `/cup` — cupping session: guided tasting, connects the cup to the roast, sets the next adjustment

You can also talk naturally at any time — "what was my last Oaxaca roast like?" or "log yellowing at 7:30" — and Claude will handle it.

## Data

```
data/
  roasters.json       # equipment catalog
  beans.json          # green bean catalog
  roasts/             # one JSON file per roast session
```

Each roast file tracks settings, timeline events (yellowing, first crack, drop), sensory notes, weight loss, cupping results, and a hypothesis loop — so you can adjust one variable per roast and trace what changed.

## Requirements

- [Claude Code](https://claude.ai/code) (the CLI or desktop app)
- A Claude account

## Roadmap

Milestone 1 is the Claude prototype — validating the data model through real roasting sessions. Milestone 2 is a web app with public profiles, bean search, and a community discovery layer.

## License

MIT
