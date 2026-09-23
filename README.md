# SyncSo — things to do in New York

An [Agent Skill](https://agentskills.io): hand `SKILL.md` to an assistant and
it can search a live catalogue of New York events, shows, classes, tours,
markets, restaurants, bars and museums — with times, prices, booking links
and pictures.

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

## The file

[`SKILL.md`](SKILL.md) is here to be read, forked and diffed — the same file
syncso.com serves, byte for byte, published to both in one step from the MCP
server's own instructions and tool definitions. So what an agent reads is
what the server actually does, and a check fails the publish if the two ever
differ.

Found something wrong in it? Open an issue here.
