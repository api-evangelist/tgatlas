---
name: Resolve and profile a Telegram channel
description: Turn a public @username into a peer id, then read the channel's profile and recent posts with per-post views and forwards.
api: tgAtlas Telegram Channel Data API
generated: '2026-09-16'
method: generated
source: openapi/tgatlas-openapi.json
operations:
  - resolveUsername
  - getFullChannel
  - getChannelMessages
---

# Resolve and profile a Telegram channel

Read a public Telegram channel starting from its @handle.

## Prerequisites

- A RapidAPI subscriber key (free tier, no card): sign up at
  https://rapidapi.com/starnikovoleg/api/telegram155
- Send both headers on every request: `X-RapidAPI-Key` and
  `X-RapidAPI-Host: telegram155.p.rapidapi.com`.

## Steps

1. **Resolve the handle.** `GET /v1/usernames/{username}` (`resolveUsername`)
   returns the numeric peer id. This spends one discovery **lookup**.
2. **Read the profile.** `GET /v1/channels/{peer_id}` (`getFullChannel`) returns
   size, description, creation date, verification/scam flags and (in the full
   chat) whether participants can be viewed. Spends a request, no lookup.
3. **Walk recent posts.** `GET /v1/channels/{peer_id}/messages`
   (`getChannelMessages`), or `GET /v1/peers/{peer_id}/history` (`getHistory`)
   for the full feed. Each post carries `views` and `forwards`. Page with
   `cursor` + `limit` (max 100; a larger limit is clamped, not rejected).

## Rules

- If a peer id is unknown to the service you get `409 RESOURCE_UNAVAILABLE` —
  seed it via `resolveUsername` first (the `hint` field says so).
- Watch `x-ratelimit-lookups-remaining` and `x-ratelimit-requests-remaining`;
  on `429 RATE_LIMITED` back off until the `-reset` header.
- A channel may own several handles: when `username` is empty, read
  `usernames[]` and use the entry that is both active and editable.
