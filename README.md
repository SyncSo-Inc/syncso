# SyncSo — things to do in New York

`SKILL.md` in this repo is an [Agent Skill](https://agentskills.io): hand it
to an assistant and it can search a live catalogue of New York events, shows,
classes, tours, markets, restaurants, bars and museums — with times, prices,
booking links and pictures.

## Use it

Send this to Claude, Claude Code, Cursor, or your own agent:

```
set up https://syncso.com/SKILL.md
```

A client that speaks MCP can skip the file and connect straight to the
endpoint:

```
https://rtdb.syncso.com/partner/mcp
```

Either way the assistant is told to authenticate, and the person it is
helping gets a code from [syncso.com/connect](https://syncso.com/connect) to
paste back. Searches are billed to that person's own account, not to whoever
wrote the assistant.

## About this copy

`SKILL.md` here is **published, not authored**. It is generated from the MCP
server's own instructions and tool definitions, so what an agent reads is
what the server actually does — the two cannot drift apart by being edited in
different places.

Read it, fork it, diff it. To report something wrong with it, open an issue;
the fix lands upstream and arrives here on the next publish.
