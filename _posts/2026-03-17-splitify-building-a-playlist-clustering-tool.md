---
title: "Splitify: Building a Playlist Clustering Tool"
date: 2026-03-14
categories: [project-writeups]
tags: [splitify, spotify, python, react, clustering]
---

Splitify breaks one large playlist into several smaller playlists based on track audio features. I built it because my playlists had turned into broad song dumps that worked poorly on shuffle. Songs I liked were still there, but the listening experience was inconsistent enough that I skipped tracks more than I enjoyed them.

The goal was to keep the songs I liked while reorganizing them into groups that actually felt cohesive.

## Why Audio Features

I chose not to split playlists only by artist or genre because those labels are still too broad in practice:

- An artist can sound very different across albums and eras.
- Genres hide a lot of variance once playlists get large.
- Two songs can share an artist or genre label while feeling completely different in energy, mood, and pacing.

Instead, I clustered tracks using audio features:

- `energy`
- `danceability`
- `valence`
- `tempo`
- `acousticness`
- `instrumentalness`
- `speechiness`
- `liveness`
- `loudness`

This also gave users room to tune the output. If the default weighting did not match their taste, they could adjust the feature weights and regenerate the split.

## Clustering Design

The core clustering algorithm is a Gaussian Mixture Model (GMM).

I used GMM because music similarity is not perfectly separable, and I wanted a model that could represent softer boundaries between groups. A rigid partitioning algorithm produced clusters that were easier to compute, but less aligned with how playlists actually feel.

High-level flow:

1. Standardize audio feature rows.
2. Apply user-adjusted feature weights after scaling.
3. Evaluate multiple candidate cluster counts.
4. Score each candidate using a weighted objective that balances compactness, tail cohesion, inter-cluster uniqueness, BIC, and cluster balance.
5. Post-process the winning result by merging tiny clusters and optionally splitting overly broad ones when cohesion improves.

I also capped output complexity with limits like maximum cluster count and minimum cluster size so the generated playlists stayed usable.

## API Constraints and Fallbacks

The hardest engineering problem was external dependency reliability.

When I started the project, Spotify's API was more practical for this workflow. Later, access to key endpoints became more restricted and effectively tied to Extended Quota approval. That created a circular dependency: I needed a working demo to justify broader access, but broader access was needed to deliver the full demo behavior.

To keep the project moving, I built a fallback path around ReccoBeats for audio features.

That pipeline worked like this:

1. Fetch ReccoBeats features in batches.
2. For unresolved tracks, fetch Spotify metadata.
3. Search Spotify for likely alternate track matches.
4. Re-query ReccoBeats for those candidate IDs.
5. If a candidate has usable features, map them back to the original missing track.

That approach recovered a meaningful number of misses without hiding unresolved coverage gaps from the user.

## Full-Stack Implementation

The frontend is a React app with two primary flows:

- `/login` starts OAuth.
- `/input-playlist` handles playlist selection, split controls, and processing progress.

Users can choose preset split criteria or open advanced controls to tune feature weights manually. Once processing begins, the client requests a job ID, polls job status, and displays progress, completion state, and the last finished playlist.

On the backend, I handled:

- OAuth and session management
- playlist retrieval
- audio feature resolution
- clustering
- new playlist creation and population
- async job status tracking

To keep larger jobs stable, I added:

- an in-process L1 cache for feature lookups
- a Postgres-backed L2 cache for shared feature and miss records
- bounded concurrency for external calls and playlist creation
- retry and backoff behavior for transient failures
- `429` handling with `Retry-After` support

Those pieces mattered as much as the model itself. Without caching, concurrency control, and fallback handling, the product felt unreliable even when the clustering logic was correct.

## Security and Session Handling

Security controls included:

- OAuth `state` verification for CSRF protection
- explicit scope tracking
- an HttpOnly session cookie
- configurable `Secure` and `SameSite` cookie settings
- restricted CORS configuration
- standard response security headers
- HSTS on secure requests

I also added token-expiration handling so expired sessions triggered a clean re-auth path instead of ambiguous client failures.

## Deployment

The current repo deploys frontend changes to Firebase Hosting on pushes to `main`. Backend linting is automated, while backend runtime deployment is handled through Google Cloud on merges to `main`.

Repository:
[github.com/stanleygvi/splitify](https://github.com/stanleygvi/splitify)
