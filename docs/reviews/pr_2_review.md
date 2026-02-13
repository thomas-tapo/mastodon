# PR #2 Review Notes

PR reviewed: https://github.com/Tap-Media/mastodon/pull/2

## Findings

### 1) API pagination links do not match query behavior (High)

`Api::V1::MemoriesController` generates `next`/`prev` links using `max_id` and `min_id`, but `load_statuses` ignores those params and paginates by `page` only.

- Current query: `.page(params[:page]).per(...)`
- Current links: `api_v1_memories_url(pagination_params(max_id: ...))` and `...min_id...`

This causes clients that follow the Link headers to request `max_id` / `min_id` values that are never applied, so pagination can repeat results or fail to advance correctly.

### 2) User preference `on_this_day_enabled` is currently unused (Medium)

The PR introduces a user setting (`web.on_this_day_enabled`) and exposes it in preferences, but there is no enforcement in the new API endpoint or UI entry points.

As implemented, users can disable the setting but still access `/memories` and `GET /api/v1/memories` as normal.

## Suggested fixes

1. Align pagination implementation and link format:
   - either implement `max_id`/`min_id` filtering in `load_statuses`,
   - or switch Link headers to page-based pagination and ensure clients use `page`.
2. Apply `on_this_day_enabled` consistently:
   - hide/disable the Memories navigation route when disabled,
   - and/or enforce the setting server-side in `Api::V1::MemoriesController`.
