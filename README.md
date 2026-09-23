# SyncSo — things to do in New York

An [Agent Skill](https://agentskills.io): hand `SKILL.md` to an assistant and
it can search a live catalogue of New York events, shows, classes, tours,
markets, restaurants, bars and museums — with times, prices, booking links
and pictures.

## Use it

Send either address to Claude, Claude Code, Cursor, or your own agent — the
file is the same at both:

```
set up https://raw.githubusercontent.com/SyncSo-Inc/syncso-skill/main/SKILL.md
```

```
set up https://syncso.com/SKILL.md
```

A client that speaks MCP can skip the file and connect straight to the
endpoint:

```
https://rtdb.syncso.com/partner/mcp
```

Whichever route, the assistant is told to authenticate, and the person it is
helping gets a code from [syncso.com/connect](https://syncso.com/connect) to
paste back. Searches are billed to that person's own account, not to whoever
wrote the assistant.

## How it is kept true

`SKILL.md` is generated from the MCP server's own instructions and tool
definitions and published to both addresses in one step, so what an agent
reads is what the server actually does. Neither address is the original and
neither lags the other; a check fails the publish if they ever differ.

Read it, fork it, diff it. Found something wrong in it? Open an issue here.
