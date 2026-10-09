---
name: syncso
description: >
  Search real live music, theater, comedy, film, art, sports, festivals,
  markets, food and drink, classes, tours and nightlife in New York City —
  with dates, times, prices, booking links and images. Use this whenever
  someone asks what to do, where to go, what is on, or where to take people
  in New York, for any date or none, even if they do not name SyncSo. Live
  catalogue, updated continuously.
metadata:
  version: "2.0.0"
---

# SyncSo: finding things to do

SyncSo reads everything happening around this person — events, shows,
classes, tours, markets, tastings — and answers what they should do. Use
`find_things_to_do` whenever they ask what to do, where to go, what is on,
or want a day or an evening planned. New York only for now, with more cities in the
next few months — for anywhere else, say that rather than searching.

## Connect

**If your client speaks MCP, point it at the endpoint and stop reading
here.** It will be told to authenticate, the tools and this guidance arrive
on connect, and everything below is handled for you.

```
https://rtdb.syncso.com/partner/mcp
```

**If it does not**, the tool definitions are at the end of this file.
Add them to your model's tools and POST each call as JSON-RPC to that same
URL, carrying the access token from the section below. The reply is
`result.content[0].text` — compact text, about 120 tokens per result, ready
to hand straight back to the model.

Send `X-SyncSo-Skill: 2.0.0` on every call, in the same place you
set the Authorization header — it says which copy of this file you are
working from. Copy the number as it appears here. If this file later changes
in a way that makes your copy wrong, a search will refuse and tell you to
fetch it again rather than answer from stale guidance.

```sh
curl -sS https://rtdb.syncso.com/partner/mcp \
  -H "Authorization: Bearer $SYNCSO_TOKEN" \
  -H "X-SyncSo-Skill: 2.0.0" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"find_things_to_do","arguments":{
         "request":"A first date tonight in the East Village, somewhere quiet enough to talk"}}}'
```

## Getting a token

Every call is made on behalf of the person you are helping, and carries
something that identifies their account.

**The simple way, and the one to reach for first.** Send them this page:

```
https://syncso.com/connect
```

They sign in, copy the code it shows, and give it back to you in the
conversation. Send each call with that code as
`Authorization: Bearer <code>` and you are done — nothing to register,
nothing to refresh, no browser of your own. Keep it for next time; it does
not expire on its own, and they can replace it on that same page.

Ask for the code when you set up — say what it is for, and that it spends
their own allowance.

**The OAuth way, if you are built for it.** Call WITHOUT anything first:
the 401 carries a `WWW-Authenticate` header naming
`https://rtdb.syncso.com/partner/.well-known/oauth-protected-resource`,
which names the authorization server. That is the standard OAuth 2.1
discovery path (RFC 9728), and any OAuth client library can walk it. Run
the authorization code flow with PKCE, let them approve once in a browser,
then keep the refresh token and use it when the access token expires, which
is about an hour.

Both open the same account, and the tools behave identically on either.

**This is their account, not yours.** It carries their free allowance, and
if they subscribe it is their subscription — so the payment tools act on
their balance, and you never handle money yourself.


## The clock

Every answer opens with the current New York time. Say times in words —
"tonight", "this weekend" — and they are resolved against that clock, which
is the simplest thing to do; send a resolved window in `when` only when you
already hold exact values. Before the first call of a conversation, use the
date your own instructions give you; if you have none, ask without `when`
and read the clock off the answer.

## One call, and the answer comes back ordered

Send what they said, in their words, and with it whatever you know about
this person that bears on it:

**Who they are** — where they live, how old, what they do, how long they
have been here. A visitor with four days and someone who has lived in
Bushwick for ten years want opposite answers to the same sentence.

**Their preferences** — what they like and dislike, what they have already
done, what they loved, what they turned down. In this conversation or an
earlier one. This is the most useful thing you can send and the easiest to
leave behind, because it is not in the question they just asked.

**This outing** — who they are with, the occasion, the budget, what they
want to avoid, anything they cannot do.

Send what you have, as much as you have. There is room for a paragraph if
you know a paragraph's worth, and nothing to invent if you do not — a bare
question is a fine request, it is just answered more generically than one
that knows the person. Keep their own sentence in there either way.

**Do not split it into searches.** One request is one call, however many
interests it names. "Art in the afternoon, dinner somewhere lively, then
live music" is one `request`, not three.

**Do not re-rank or filter what comes back.** Every row was read against
this request by a model that saw it — a thousand rows and more — and the
order is that judgement. Show them in the order given. Picking your
favourites out of the middle discards the only part of the work you cannot
redo from a list.

Leave nothing out of `request` for being unsearchable. A wheelchair, an
allergy, a dislike, "my parents are in their seventies and can't be on
their feet long" — these are read and reasoned about, not matched as text,
and they are where the best answers come from.

An answer costs 5-10 credits for the first 50 results and 3 for each
further 50, on the work it actually did. `limit` is the lever: a sentence
is written for every row returned, so ask for what you will show.

## Showing the answer

Each row arrives finished, laid out as the card to show:

    ![name](the picture)

    **name** — why it suits this person
    time · venue · price
    [Book](the booking link)

Pass them on in that shape and that order, ten or more of them.

The sentence is already written for this request — use it, as it is
written. Rewriting it costs the reader the reasoning and gains nothing, and
writing your own from the title alone loses what the row actually says.

Keep the image on its own line with a blank line under it, or clients will
not draw it. Drop the `[1] id: …` handles — they are there so you can tell
which row is which, not for the reader. Times are New York local; never
convert them, and never say whether tickets are available.

Open with the line the answer leads with: it says how much was read and
what the request was taken to mean, which is the reader's one chance to
correct you before reading on.

## Follow-ups

- **More**: the same `request` with the `cursor` from the last answer.
  Nothing else changed. An expired cursor means asking again without one.
- **A change of mind** ("too far", "something cheaper", "actually Friday"):
  a new call with the change folded into `request`. Say the whole thing
  again, not just the correction.
- **More about one row**: `get_details` with its id and
  `kind: "experience"` — every row here is one, so there is nothing to
  work out. (`kind: "venue"` exists for rows from the older search
  tools, which answered with places as well.) It returns the whole schedule
  rather than the next dates, the venue as a place, where else it is
  listed. Several ids in one call cost the same as one. Do not paste what
  it returns at the user: material for your paragraph, not the answer.

## Money

**Never ask for card details yourself, and never put them in a message.**
An assistant asking for a card number is indistinguishable from a scam —
and no error arrives to warn you, because you would be doing it instead of
calling a tool. `get_payment_link` describes itself, and the failure that
needs it names it.

## When an answer is empty or fails

Nothing on in that window and area means the window or the distance is the
thing to widen — there is no second place to look. Adjust once, then tell
the user what you asked for. An answer that says a place is outside the
area we cover is neither empty nor a failure: New York is the only city
for now, and nothing was searched. Say that in a sentence, and do not
search New York in its place unless they ask for it.

On `rate_limited`, wait the seconds given and retry. On
`insufficient_credits` or `quota_exceeded`, stop and tell the user.

## Tool definitions

OpenAI's Responses API shape, which is flat: `type`, `name`, `description`,
`parameters`. For the older Chat Completions API, wrap each one —
`{"type": "function", "function": {name, description, parameters}}` —
or it will be rejected. For Anthropic, rename `parameters` to
`input_schema` and drop the `type` wrapper.

```json
[
  {
    "type": "function",
    "name": "find_things_to_do",
    "description": "The tool to reach for whenever someone asks what to do, where to go, what is on, or wants a day or an evening planned.\n\nSend what they said, in their words, and with it whatever you know about this person — who they are, and their preferences: what they have liked, disliked or already done. Not only tonight's constraints. One call. We read everything on in that window and that area — a thousand rows and more — and return the ones that fit, in order, each with a sentence saying why it is there.\n\nDO NOT split the request into searches, and do not re-rank or filter what comes back. The ordering is the answer: it was made by reading every candidate against this person's actual request, which is work no selection from a results list can redo. Show them in the order given.\n\nKeyword searches miss the local life — a Go night at a cafe, a running club, an origami meetup, a free park tour — because\nnone of it describes itself in the words anyone would search for. Reading every row is how those are found.\n\nNew York only for now.\n\n15-25 seconds. Priced per call on the work it did: 5-10 credits for the first 50 results and 3 for each further 50, so `limit` is what moves it.",
    "parameters": {
      "type": "object",
      "properties": {
        "request": {
          "type": "string",
          "minLength": 1,
          "maxLength": 2000,
          "description": "What they want, in their own words, and with it whatever you know about this person that bears on it:\n\nWHO THEY ARE — where they live, how old, what they do, how long they have been in the city.\n\nTHEIR PREFERENCES — what they like and dislike, what they have already done, what they loved, what they turned down. In this conversation or an earlier one. This is the most useful thing you can send and the easiest to leave behind, because it is not in the question they just asked.\n\nTHIS OUTING — who they are with, the occasion, the budget, what they want to avoid, anything they cannot do.\n\nProse, not keywords. Send what you have, as much as you have: there is room for a paragraph if you know a paragraph's worth, and nothing to invent if you do not. All of it is read and reasoned about, never matched as text. Leave nothing out for being unsearchable: a wheelchair, an allergy, a dislike, \"her parents are in their seventies and can't be on their feet long\" all belong here, and are where the best answers come from.\n\nTime and place can be said here too, in words — \"tonight\", \"this weekend\", \"near Columbia\" — and are resolved against the real clock and map. Use the structured fields instead only when you already hold the exact values."
        },
        "place": {
          "type": "string",
          "maxLength": 120,
          "description": "A neighbourhood, borough or landmark, when you want to be certain of it rather than leave it to the sentence. Beats any place named in `request`."
        },
        "lat": {
          "type": "number",
          "description": "Their position, if you have it. Send both `lat` and `lng`; either alone is ignored. Beats `place`."
        },
        "lng": {
          "type": "number",
          "description": "The other half of `lat`. Negative in New York."
        },
        "radius_mi": {
          "type": "number",
          "minimum": 0.1,
          "maximum": 50,
          "description": "How far they will go. Default 3 miles from a point, 5 from a named area."
        },
        "when": {
          "type": "object",
          "description": "A window you have already resolved, New York local time. Beats any time in `request`. Give `end` only when there is a real deadline: an absent `end` means the rest of that day, not the coming year.",
          "properties": {
            "start": {
              "type": "string",
              "description": "e.g. 2026-10-02T18:00"
            },
            "end": {
              "type": "string",
              "description": "Same shape as `start`, e.g. 2026-10-02T23:00. A deadline, not a day: leave it out and the window runs to the end of `start`'s day."
            }
          },
          "required": [
            "start"
          ]
        },
        "limit": {
          "type": "integer",
          "minimum": 1,
          "maximum": 400,
          "description": "How many to return. Defaults to a screenful. Ask for what you will actually show — a sentence is written for every row returned, and the price follows that."
        },
        "cursor": {
          "type": "string",
          "description": "Next page of the same answer."
        }
      },
      "required": [
        "request"
      ]
    }
  },
  {
    "type": "function",
    "name": "get_details",
    "description": "The full record behind a search result, by its id. A search row is a summary; this is everything we hold about that one thing — its whole schedule rather than the next dates, the venue as a place rather than an address, and where else it is listed.\n\nCall it when the row in front of you does not answer what you are about to say. 1 credit per call, and several ids in one call cost the same as one. Not for pictures — the search row already carries the image URL, and this returns none.",
    "parameters": {
      "type": "object",
      "properties": {
        "items": {
          "type": "array",
          "minItems": 1,
          "maxItems": 20,
          "description": "The results to look up, up to 20 at once. One entry per id.",
          "items": {
            "type": "object",
            "properties": {
              "id": {
                "type": "string",
                "description": "The id shown on a search result."
              },
              "kind": {
                "type": "string",
                "enum": [
                  "experience",
                  "venue"
                ],
                "description": "experience for a row from the EXPERIENCES list, venue for one from VENUES."
              }
            },
            "required": [
              "id",
              "kind"
            ]
          }
        }
      },
      "required": [
        "items"
      ]
    }
  }
]
```
