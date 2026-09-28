# SyncSo

**Let your agents understand what's happening locally, in real time.**

Ask a model what to do in New York and it names the Whitney. It is not wrong.
It is just answering from a guidebook, because that is what it has — a city as
a list of places, frozen whenever the training run ended.

What it does not have is everything going on inside those places, and the
thousand rooms that never make a guidebook. The Tuesday gig. The pottery class
with two seats left. The night market under the bridge. Your model has never
heard of any of it.

SyncSo is that missing half: live music, theater, comedy, film, art, sports,
festivals, markets, food and drink, classes, tours, nightlife — the whole
surface of a city, each thing carrying its dates, times, prices, booking links
and images. Not one category. Not the famous parts.

Which means your agent can finally take the questions it used to hand back.
Where to take a visiting parent on a Tuesday. What a couple could book three
weeks out. What is free, outdoors, and happening in the next four hours.
Real answers, with a link to book them.

## Install

Tell your assistant to set itself up from this file:

```
Set up SyncSo from https://syncso.com/SKILL.md
```

That is the whole thing. The file carries the guidance, the tool definitions
and how to get a token, and it is always the current copy. Your assistant reads
it and wires itself up. Nothing to register, and no key of yours to manage —
each user signs in once at [syncso.com/connect](https://syncso.com/connect) and
the calls run on their own allowance.

Works with anything that can call a tool and make an HTTPS request. A client
that speaks MCP can skip the file and connect straight to
`https://rtdb.syncso.com/partner/mcp` instead.

Two tools, `search_directions` and `get_details`. That is the whole surface.

## Coverage

New York only, for now. More cities over the coming months.

Within it, everything: every category above, from the institutions down to the
one-night thing in a room above a bar. Ask about anywhere else and the skill
tells your model to say so rather than invent an answer — the same refusal to
guess that makes the New York answers worth trusting.

## About this file

Nobody writes [`syncso/SKILL.md`](syncso/SKILL.md) by hand. It is generated
from the server's own instructions and tool definitions, and published here and
to [syncso.com/SKILL.md](https://syncso.com/SKILL.md) in the same step — so
what you read is what the server is still saying, not what it said in March.

## Links

- [syncso.com](https://syncso.com) — what SyncSo is
- [syncso.com/connect](https://syncso.com/connect) — where your user gets their code
- [syncso.com/SKILL.md](https://syncso.com/SKILL.md) — this skill, always current

## Questions

Something missing, wrong, or not working the way this page says? Open an
[issue](https://github.com/SyncSo-Inc/syncso/issues), or email
[hello@syncso.com](mailto:hello@syncso.com). Tell us if a city you need is not
here yet — that is useful to know.

## License

MIT. See [LICENSE](LICENSE).
