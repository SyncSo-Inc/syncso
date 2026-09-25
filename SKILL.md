---
name: syncso
description: >
  Search real live music, theater, comedy, film, art, sports, festivals,
  markets, classes, tours and nightlife in New York City, and the
  restaurants, bars, museums and venues around them — with dates, times,
  prices, booking links and images. Use this whenever someone asks what to
  do, where to go, what is on, or where to eat, drink or take people in New
  York, for any date or none, even if they do not name SyncSo. Live
  catalogue, updated continuously.
metadata:
  version: "1.0.1"
---

# SyncSo: finding things to do

SyncSo scans thousands of things happening around this person every day —
events, shows, classes, tours, markets, restaurants, bars, museums — so you
can find the few that are right for them. Use `search_directions` whenever
they ask what to do, where to go, what is on, or want a plan. New York only
for now, with more cities in the next few months — for anywhere else, say
that rather than searching.

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

Send `X-SyncSo-Skill: 1.0.1` on every call, in the same place you
set the Authorization header — it says which copy of this file you are
working from. Copy the number as it appears here. If this file later changes
in a way that makes your copy wrong, a search will refuse and tell you to
fetch it again rather than answer from stale guidance.

```sh
curl -sS https://rtdb.syncso.com/partner/mcp \
  -H "Authorization: Bearer $SYNCSO_TOKEN" \
  -H "X-SyncSo-Skill: 1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"search_directions","arguments":{
         "queries":["live jazz","comedy","gallery opening"],
         "location":{"city":"New York"}}}}'
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


Every search result opens with the current New York time. Build time
windows from that clock — tonight is today 18:00-23:59, this weekend is the
coming Saturday and Sunday. Before the first search of a conversation, use
the date your own instructions give you; if you have none, search without
`time_windows` and read the clock off the result.

## Split the request into directions

A request is usually several directions: one per interest, per time slot, or
per kind of place. "Art in the afternoon, dinner somewhere lively, then live
music" is three. They all go in one call.

Five is typical, and more directions are available. Where the person
named fewer, fill the rest — with the obvious neighbours of what they
asked for, or, pinned down to a single subject, with different wordings of
it. A blank entry searches whatever is simply on, which is the honest
direction when you would otherwise be inventing one.

Before you search, summarise what you know about this person, and think
about how to break that into directions. Search for the person, not the
question: what you know goes into the wording and the filters, since there
is no profile field. What kind of occasion it is belongs in the query, as
the atmosphere they would recognise; what is answerable as yes-or-no
belongs in a filter, and two exist — `is_free` and `environment_types`.
Keep each query to a few words, not a paragraph about them. What no search
can express, apply yourself when choosing.

## Before you answer

Write like a person who went and looked, not like a program printing
records. A row is your material, not your answer: turn it into prose, and
drop the field names, the ids and anything they cannot act on. That is a
rule about VOICE, not about how much to show — say as much as is useful.

Whenever replying, give ten or more results; don't be less than ten. Say
how many SyncSo found, so they know what you chose from, but not how many
searches you ran.

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

Named no direction at all? Send no `queries` and see what is on — a real
answer rather than a guess, and something concrete to offer a choice from.
Ask a question only if that comes back thin.

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

What they want decides the call, not how they phrase it. Before any of
these, look at the results you already have and have not shown; if they are
useful, consider reusing them first and add the following at your
discretion.

- **More of the same**: the batch again with its `cursor`, nothing else
  changed — every direction moves forward together. An expired cursor
  means running it again without one.
- **A different direction**: a new search — a cursor only continues the
  batch that made it.
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
    "name": "search_directions",
    "description": "THE SEARCH TO REACH FOR. Several directions in one call — music, and comedy, and something outdoors — sharing one place and one set of times, answered together in 2-4 seconds.\n\nFive is typical, and more directions are available — not a budget to spend carefully. A person who asked for live music still wants to hear what else is on that night, and one who named nothing needs the directions to BE your answer to what they might like — summarise what you know about them and split that into directions. Pinned down to a single subject, send several wordings of it rather than one.\n\nExperiences only — this tool does not find bars or restaurants to sit in.\n\n1 credit per 20 results in each direction, so five directions of twenty is five credits — the same as running the five searches separately.\n\nSearch for the person, not the question: put what you know about them — tastes, budget, neighborhood, who they are with, what they avoid — into the wording of each direction and into the filters; there is no profile field. Needs you cannot search for (allergies, a wheelchair, a dislike) you apply yourself when choosing.\n\nThen write up what you found — ten or more unless they asked for a short answer — laying each one out the way the connect-time instructions show: picture, name and why it suits this person, then time, venue and price, then the booking link. Order by what matters most to them, not by the direction it arrived under. Times shown are New York local — never convert them, and never say whether tickets are available.",
    "parameters": {
      "type": "object",
      "properties": {
        "queries": {
          "type": "array",
          "minItems": 5,
          "items": {
            "type": "string",
            "minLength": 1,
            "maxLength": 1000
          },
          "description": "5 is typical, and more are available. Each in simple words and phrases rather than a sentence. Keep each one short and general — a narrow wording has fewer good things to choose from. Put the time in time_windows and the place in location, not here.\n\nNot a budget to spend carefully: even one subject is worth several, because different wordings reach different things. Asked for live music, send several ways of saying it rather than one. Where the person gave you fewer, fill the rest with what else they might like.\n\nAn entry may be an empty string, which searches whatever is simply on — the honest direction when you would otherwise be inventing one. Omit `queries` altogether and the whole call becomes that."
        },
        "cursor": {
          "type": "string",
          "description": "For 'more like these' only: the cursor from a previous result, for the next page of the SAME batch — every direction moves forward together, with every other argument unchanged. Nothing repeats and the order does not shift. Expires after 15 minutes; if expired, run the original search again without it. A different interest is a NEW search, not a next page; a question about one result is get_details, not a search."
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
        "limit": {
          "type": "integer",
          "minimum": 20,
          "maximum": 60,
          "description": "Results per direction. Start at 20: anything up to 20 costs the same single credit for that direction, so asking for 10 buys half as much for the same price."
        },
        "is_free": {
          "type": "boolean",
          "description": "true keeps only free experiences, false only paid ones. Rows with unknown price are left out either way. Use for a tight budget. Applies to every direction."
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
          "description": "Keep only experiences in these settings: indoor, outdoor, mixed. Use for weather. Applies to every direction."
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
        }
      },
      "required": [
        "location"
      ]
    }
  },
  {
    "type": "function",
    "name": "search_experiences",
    "description": "ONE direction, and the only search that can return venues — standing places with no event attached, when the user wants somewhere to sit rather than something to attend. For the usual several-directions answer use `search_directions` instead; reach for this one when a single follow-up is all that is left to ask, or when the answer is a place.\n\nExperiences in New York — events, shows, classes, tours, markets — and the places they happen in. 2-4 seconds; 1 credit per 20 results, up to 60 per search. Left to defaults: 20 results per type, both experiences and venues, every upcoming date, whole city, no filters.\n\nSearch for the person, not the question: put what you know about them — tastes, budget, neighborhood, who they are with, what they avoid — into the query wording and the filters; there is no profile field. Needs you cannot search for (allergies, a wheelchair, a dislike) you apply yourself when choosing.\n\nThen write up what you found — ten or more unless they asked for a short answer — laying each one out the way the connect-time instructions show: picture, name and why it suits this person, then time, venue and price, then the booking link. Order by what matters most to them. Times shown are New York local — never convert them, and never say whether tickets are available.",
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "string",
          "maxLength": 1000,
          "description": "One direction, in simple words and phrases rather than a sentence. Keep it short and general — a narrow query has fewer good things to choose from. Put the time in time_windows and the place in location, not here.\n\nOmit it entirely to see what is simply on — the remaining constraints become the ask. That is the honest search when the user has named no direction at all and you would otherwise be inventing one for them. It is a starting point, not a shortcut: once they have said what they are after, one search per direction beats one without. Send no `query` rather than an empty string; a blank one is refused."
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
          "minimum": 20,
          "maximum": 60,
          "description": "Results for this search, per result type. Start at 20: anything up to 20 costs the same single credit, so asking for 10 buys half as much for the same price."
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
