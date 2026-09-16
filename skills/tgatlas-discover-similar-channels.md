---
name: Discover similar channels by walking the recommendation graph
description: Start from one known channel and expand outward through Telegram's own similar-channel recommendations to reach channels whose names you do not already know.
api: tgAtlas Telegram Channel Data API
generated: '2026-09-16'
method: generated
source: openapi/tgatlas-openapi.json
operations:
  - getChannelRecommendations
  - getFullChannel
  - searchContacts
---

# Discover similar channels

Find channels you cannot name yet by walking Telegram's own topical graph.

## Prerequisites

- RapidAPI key + both gateway headers (see the resolve-and-profile skill).

## Steps

1. **Find a seed.** If you do not have a starting channel, use
   `GET /v1/contacts/search` (`searchContacts`) to search the public directory.
   Spends a lookup.
2. **Expand.** `GET /v1/channels/recommendations`
   (`getChannelRecommendations`) returns the channels Telegram recommends
   alongside a seed — a topical judgement, not a keyword match. Spends a lookup.
3. **Enrich each result.** For each returned channel call
   `GET /v1/channels/{peer_id}` (`getFullChannel`) to attach exact subscriber
   counts and posting activity. One request per channel; no lookup.
4. **Iterate.** Feed promising results back into step 2. Two rounds of expansion
   from a few dozen seeds reaches several thousand channels.

## Rules

- Recommendation and search routes spend the metered **lookups** counter;
  profile reads do not — favour reads when fanning out to control cost.
- De-duplicate peer ids across rounds before re-expanding.
- Respect `x-ratelimit-lookups-remaining`; on `429` back off until `-reset`.
