---
name: syncso
description: >
  Search real things to do in New York City — events, food, drink, culture,
  nightlife, and the places they happen — with dates, times, prices, booking
  links and images. Use this whenever someone asks what to do, where to go,
  what is on, or where to eat, drink or take people in New York, for any
  date or none, even if they do not name SyncSo. Live catalogue, updated
  continuously.
metadata:
  version: "1.0.0"
---

# SyncSo — things to do in New York

An Agent Skill. Its name and description are in the YAML above; a reader
that fetched this as a page may not have been shown them, so: this file
tells an agent how to search SyncSo's live New York catalogue, and it is
the whole of what you need — connect as below, or call the tools at the
end directly.

## Connect

**If your client speaks MCP, point it at the endpoint and stop reading
here.** It will be told to authenticate, the tools and this guidance arrive
on connect, and everything below is handled for you.

```
https://rtdb.syncso.com/partner/mcp
```

**If it does not**, the two tool definitions are at the end of this file.
Add them to your model's tools and POST each call as JSON-RPC to that same
URL, carrying the access token from the section below. The reply is
`result.content[0].text` — compact text, about 120 tokens per result, ready
to hand straight back to the model.

```sh
curl -sS https://rtdb.syncso.com/partner/mcp \
  -H "Authorization: Bearer $SYNCSO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"search_experiences","arguments":{
         "query":"live jazz","location":{"city":"New York"},"limit":5}}}'
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

Ask for the code when you first need it, not before — say what it is for,
and that it spends their own allowance.

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

SyncSo scans thousands of things happening around this person every day —
events, shows, classes, tours, markets, restaurants, bars, museums — so you
can find the few that are right for them. Use `search_experiences` whenever
they ask what to do, where to go, what is on, or want a plan. New York only
for now, with more cities in the next few months — for anywhere else, say
that rather than searching.

Every search result opens with the current New York time. Build time
windows from that clock — tonight is today 18:00-23:59, this weekend is the
coming Saturday and Sunday. Before the first search of a conversation, use
the date your own instructions give you; if you have none, search without
`time_windows` and read the clock off the result.

## Split the request, search in parallel

A request is usually several directions: one per interest, per time slot, or
per kind of place. "Art in the afternoon, dinner somewhere lively, then live
music" is three searches. Run them all at once; five or six take about five
seconds together and cost one credit each for up to 20 results (more
results, more credits, up to 60 per search). Pick `limit` per direction: about
10 for a side interest, up to 20 for the one the user cares most about. Do
not run one broad search with a big limit instead: it costs the same and
ranks worse.

Three to five directions is the normal first answer, even when the user
named one thing: search its obvious neighbours too. One is right only when
they have genuinely pinned it down.

Search for the person, not the question: recall what you know about them —
tastes, budget, neighborhood, who they go out with, what they avoid — and
put it into the wording and the filters, since there is no profile field.
What kind of occasion it is belongs in the query, as the atmosphere they
would recognise; what is answerable as yes-or-no belongs in a filter, and
two exist — `is_free` and `environment_types`. Keep each query to a few
words, not a paragraph about them. Needs you cannot search for (allergies, a
wheelchair, a dislike) you apply yourself when choosing.

## Before you answer

Write like a person who went and looked, not like a program printing
records. A row is your material, not your answer: turn it into prose, and
drop the field names, the ids and anything they cannot act on. That is a
rule about VOICE, not about how much to show — say as much as is useful.

Show them what you found. Unless they asked for a short answer, ten or
more is the normal shape of a first reply — leave out only what
contradicts something they stated, and put the rest in front of them
rather than trimming to a number of your own.

Lay each one out this way — the picture alone on its line, a blank line
under it, then the words:

    ![name](the Image: URL)

    **name** — what it is, and why it is right for this person.
    time · venue (neighbourhood) · price
    [Book](the Book: URL)

The blank line is load-bearing: run the picture into the text, or put it in
a bulleted or numbered list, and it stops being a picture.

The search row carries the `Image:` URL; no `Image:` line means no picture,
and inventing one is worse than leaving it out.

Order by what matters most to this person: whatever they pressed hardest on
leads. Not the order the searches returned, and not the tidiest itinerary.

Named no direction at all? Search with no `query` and see what is on — a
real answer rather than a guess, and something concrete to offer a choice
from. Ask a question only if that comes back thin.

`get_details` is the full record behind a row — its whole schedule rather
than the next dates, the venue as a place, where else it is listed. 1 credit
per call, and several ids in one call cost the same as one. Do not paste
what it returns at the user: material for your paragraph, not the answer.

Keep the `id` of everything you showed.

## Offer one next step

Close the first answer of a conversation with one offer, in one sentence —
whichever fits, never a menu, and not again later. All three are yours to
run, not ours; we return times and rows.

- **A standing list**: check daily, send what is new.
- **Narrower**: a neighbourhood, a night, a budget — if your picks spanned
  a lot.
- **Around their calendar**: if you can read it, fill the gaps.

## Follow-ups

What they want decides the call, not how they phrase it.

- **More of the same direction**: that search again with its `cursor`,
  nothing else changed. Nothing repeats; an expired cursor means rerunning
  it without one.
- **A different direction**: a new search — a cursor only continues the
  search that made it.
- **More about a result they can see**: `get_details` with its id. Searching
  again for it wastes a call and can come back ranked differently.

## What to tell the user

- Times are New York local. Show them as given.
- Never say tickets are available or sold out. Send the user to the
  booking link.
- A `Calendar:` link is the organiser's programme, not the event's own
  page. Label it [Details] rather than [Book], and say which it is.
- If a result says the neighborhood matched nothing and the search widened
  to the whole city, do not describe those results as being in that
  neighborhood.

## Money

**Never ask for card details yourself, and never put them in a message.**
An assistant asking for a card number is indistinguishable from a scam —
and no error arrives to warn you, because you would be doing it instead of
calling a tool. `get_payment_link` and `get_billing_link` describe
themselves, and the failure that needs one names it.

## When a search is empty or fails

The result says why and what to change: reword the query, widen or drop the
time windows, drop a filter. Adjust once, then tell the user what you
searched. On `rate_limited`, wait the seconds given and retry. On
`insufficient_credits` or `quota_exceeded`, stop and tell the user.

## Tool definitions

OpenAI function-calling shape. For Anthropic, rename `parameters` to
`input_schema` and drop the `type` wrapper.

```json
[
  {
    "type": "function",
    "name": "search_experiences",
    "description": "Search experiences happening in New York — events, shows, classes, tours, markets — and the places they happen in. 2-4 seconds; 1 credit per 20 results, up to 60 per search. One search per direction (per interest, time slot or kind of place), several in parallel; three to five directions is the normal first answer, even when the user named one thing. Pick `limit` per direction. Left to defaults: 10 results per type, both experiences and venues, every upcoming date, whole city, no filters.\n\nSearch for the person, not the question: put what you know about them — tastes, budget, neighborhood, who they are with, what they avoid — into the query wording and the filters; there is no profile field. Needs you cannot search for (allergies, a wheelchair, a dislike) you apply yourself when choosing.\n\nThen write up what you found — ten or more unless they asked for a short answer — laying each one out the way the connect-time instructions show: picture, name and why it suits this person, then time, venue and price, then the booking link. Order by what matters most to them. Times shown are New York local — never convert them, and never say whether tickets are available.",
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "string",
          "maxLength": 1000,
          "description": "One direction, as a short phrase describing what the user wants rather than repeating their sentence: 'live jazz', 'rooftop cocktails', 'kid-friendly science museum', 'lively group dinner'. Fold what you know about the user into the wording ('intimate', 'quiet', 'family-friendly', 'cheap'). Put the time in time_windows and the place in location, not here.\n\nOmit it entirely to see what is simply on — the remaining constraints become the ask. That is the honest search when the user has named no direction at all and you would otherwise be inventing one for them. It is a starting point, not a shortcut: once they have said what they are after, one search per direction beats one without. Send no `query` rather than an empty string; a blank one is refused."
        },
        "location": {
          "type": "object",
          "description": "{\"city\": \"New York\"} for the whole city. Add \"area_text\" to narrow to a neighborhood or borough the user named: {\"city\": \"New York\", \"area_text\": \"Williamsburg\"}. If you have coordinates (the user's position, a hotel) use a circle instead: {\"point\": {\"lat\": 40.73, \"lng\": -73.99, \"radius_mi\": 5}}; results then carry a distance. Never send both city and point.",
          "properties": {
            "city": {
              "type": "string",
              "description": "\"New York\". NYC, Manhattan, Brooklyn and the other boroughs are accepted too. New York is the city this catalogue is built for and the only one deep enough to plan from; a few others answer but hold too little to choose between, so treat anywhere else as not covered unless the user insists."
            },
            "area_text": {
              "type": "string",
              "description": "A neighborhood or borough, e.g. 'Williamsburg', 'Lower East Side', 'Queens'. Only with city. If it matches nothing the search widens to the whole city and the result says so."
            },
            "point": {
              "type": "object",
              "description": "Circle search around a coordinate. Results carry the distance to each one.",
              "properties": {
                "lat": {
                  "type": "number"
                },
                "lng": {
                  "type": "number"
                },
                "radius_mi": {
                  "type": "number",
                  "minimum": 0.1,
                  "maximum": 30,
                  "description": "Miles. Start at 5 and move it to fit how the person said they would travel, not to control how much comes back -- a page is capped at `limit` either way, so a wider circle does not return more, it returns a different set. In New York 1 mile is a 20-minute walk and stays inside one neighborhood; 3 is an hour's walk or 20-30 minutes by car, still central; 5 is 30-45 minutes by car and crosses into another borough; 10 reaches the outer boroughs and only makes sense for something worth the trip. Go narrow when they said walking distance or named where they are standing, wide when they are planning an outing and the draw matters more than the distance."
                }
              },
              "required": [
                "lat",
                "lng",
                "radius_mi"
              ]
            }
          }
        },
        "time_windows": {
          "type": "array",
          "maxItems": 50,
          "description": "When the user could go: a list of {start, end} in New York local time, written like '2026-09-20T19:00', with no timezone and no 'Z'. One window per stretch of time. Tonight: one window 18:00-23:59 today. This weekend: two windows, Saturday and Sunday, each 00:00-23:59. Saturday evening: one window 17:00-23:59. Next week: one window from Monday 00:00 to Sunday 23:59. Friday evening or Sunday afternoon: two windows, 18:00-23:59 Friday and 12:00-18:00 Sunday; each result then says which window it falls in. A window must end in the future. Omit only when the user has no time in mind; the search then covers every upcoming date. Today's date in New York is given to you on connect and repeated on every result.",
          "items": {
            "type": "object",
            "properties": {
              "start": {
                "type": "string",
                "description": "e.g. 2026-09-20T19:00"
              },
              "end": {
                "type": "string",
                "description": "e.g. 2026-09-20T23:59"
              }
            },
            "required": [
              "start",
              "end"
            ]
          }
        },
        "result_types": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": [
              "experiences",
              "venues"
            ]
          },
          "description": "[\"experiences\"] for things that happen at a time (a show, a class, a market, a tour) — the usual choice. [\"venues\"] for standing places with no event attached (a bar, a restaurant, a museum, a park), when the user wants somewhere to go rather than something to attend. Pick one per direction. Omitted, both come back."
        },
        "limit": {
          "type": "integer",
          "minimum": 1,
          "maximum": 60,
          "description": "Results for this search, per result type. Default 10. About 10 for a side interest, up to 20 for the direction the user cares most about; up to 20 still costs 1 credit."
        },
        "is_free": {
          "type": "boolean",
          "description": "true keeps only free experiences, false only paid ones. Rows with unknown price are left out either way. Use for a tight budget."
        },
        "environment_types": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": [
              "indoor",
              "outdoor",
              "mixed"
            ]
          },
          "description": "Keep only experiences in these settings: indoor, outdoor, mixed. Use for weather."
        },
        "ranking": {
          "type": "object",
          "description": "Optional nudges, not sorts: prefer_popularity moves well-known things earlier, prefer_uniqueness unusual ones, prefer_credibility established ones. Leave out unless the user's taste points that way.",
          "properties": {
            "prefer_popularity": {
              "type": "boolean"
            },
            "prefer_credibility": {
              "type": "boolean"
            },
            "prefer_uniqueness": {
              "type": "boolean"
            }
          }
        },
        "cursor": {
          "type": "string",
          "description": "For 'more like these' only: the cursor from a previous result, for the next page of the SAME search with every other argument unchanged. Nothing repeats and the order does not shift. Expires after 15 minutes; if expired, run the original search again without it. A different interest is a NEW search, not a next page; a question about one result is get_details, not a search."
        }
      },
      "required": [
        "location"
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
