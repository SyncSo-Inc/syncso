# SyncSo

**The agent skill for knowing what is actually on tonight.**

Ask a model what to do in New York and it will give you the Village Vanguard,
the Whitney, and a stroll on the High Line — the same landmarks it would have
named three years ago, because that is what a training set remembers. It does
not know the Vanguard's second set is sold out, that the Whitney is closed
today, or that the thing this person would actually love is a 9pm show in
Bushwick by a band they have never heard of.

SyncSo is the missing half. It crawls thousands of things happening across
New York every day — shows, gigs, classes, tours, markets, tastings, openings
— and keeps dates, times, prices, booking links and images current. One call
gives your model live material to choose from instead of a guess to
apologise for.

```
https://rtdb.syncso.com/partner/mcp
```

Supports **Claude Code**, **Claude Desktop**, and any MCP client. Not built
for MCP? The tool definitions ship in
[`syncso/SKILL.md`](syncso/SKILL.md) and the endpoint speaks plain JSON-RPC
over HTTPS — the same two calls work from anything that can POST.

---

## What it gives your agent

- **Several questions in one call.** "Art in the afternoon, dinner somewhere
  lively, then live music" is three directions, answered together in 2-4
  seconds — not three round trips.
- **Answers already shaped for a reply.** Results come back as compact text,
  roughly 120 tokens each, with the card layout to render them and a picture
  URL that is safe to hotlink. You hand them to your model, not to a parser.
- **Enough to actually choose from.** Twenty results per direction for one
  credit, so the model picks the right few instead of reading out the only
  three it got.
- **The details when a row is not enough** — full schedules, the venue as a
  place, where else a thing is listed.
- **Time that is always now.** Every result opens with the current New York
  clock, so "tonight" and "this weekend" resolve correctly without you
  putting a clock in your system prompt.

Two tools, `search_directions` and `get_details`. That is the whole surface.

## Getting a token

Every call is made on behalf of the person your agent is helping, and spends
their allowance — not yours. You never handle their money.

**The simple way.** Send them to:

```
https://syncso.com/connect
```

They sign in, copy the code, and paste it back into the conversation. Send it
as `Authorization: Bearer <code>` and you are done — nothing to register, no
refresh, no browser of your own. It does not expire on its own.

**The OAuth way**, if your client is built for it: call with no credentials
and the `401` carries a `WWW-Authenticate` header naming the standard
discovery document (RFC 9728). Any OAuth 2.1 library can walk it. Both routes
open the same account and behave identically.

## Install

### Claude Code

```bash
claude plugin marketplace add SyncSo-Inc/syncso
claude plugin install syncso@syncso
```

Or drop the skill in by hand:

```bash
git clone https://github.com/SyncSo-Inc/syncso /tmp/syncso

# Available in every project
mv /tmp/syncso/syncso ~/.claude/skills/syncso

# Or just this project
mv /tmp/syncso/syncso .claude/skills/syncso
```

Keep the directory named `syncso` — the skill spec requires it to match the
`name` in the file's frontmatter.

### Any MCP client

Point it at `https://rtdb.syncso.com/partner/mcp`. The tools and the full
usage guidance arrive on connect, so there is nothing else to install and
nothing in this repository you need.

### Everything else

Read [`syncso/SKILL.md`](syncso/SKILL.md). Its last section is the tool
definitions in OpenAI function-calling shape (rename `parameters` to
`input_schema` for Anthropic), and each call is a JSON-RPC POST to the same
URL. One `curl` in that file shows the whole shape.

## Try it

```bash
curl -sS https://rtdb.syncso.com/partner/mcp \
  -H "Authorization: Bearer $SYNCSO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"search_directions","arguments":{
         "queries":["live jazz","comedy","gallery opening"],
         "location":{"city":"New York"}}}}'
```

## Coverage

New York, in depth, right now. More cities over the coming months. Ask about
anywhere else and the skill will tell your model to say so rather than invent
an answer — which is the point of the whole thing.

## About this file

[`syncso/SKILL.md`](syncso/SKILL.md) is generated from the MCP server's own
instructions and tool definitions and published here and to
[syncso.com/SKILL.md](https://syncso.com/SKILL.md) in one step. Neither copy
is the original, and neither is edited by hand — so guidance you read here is
guidance the server is still giving.

## Links

- [syncso.com](https://syncso.com) — what SyncSo is
- [syncso.com/connect](https://syncso.com/connect) — where your user gets their code
- [syncso.com/SKILL.md](https://syncso.com/SKILL.md) — this skill, always current

## License

MIT. See [LICENSE](LICENSE).
