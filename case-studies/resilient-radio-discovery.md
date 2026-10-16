# Resilient Radio Discovery

How my local-first radio logger separates optional radio-source outages from local workflow failures.

- **Source:** [Hardcore Radio Logger](https://github.com/wouthh/hardcore-radio-logger), a Python and SQLite project I designed and built. The fallback described here arrived in [PR 6](https://github.com/wouthh/hardcore-radio-logger/pull/6).

## Summary

The poller asks the configured Icecast status endpoint for track metadata first. When Icecast is unavailable or returns no usable track, it tries the broadcaster's official player page. Each new poll starts with Icecast again, so the fallback never becomes a permanent source preference.

The important boundary is what happens when neither source has usable metadata. A standalone `poll-radio` call fails explicitly. The larger `run-once` workflow records a warning and continues its later stages without adding a radio observation. In verbose apply mode it also records an unavailable-poll event with sanitised failure reasons. Local file, logger, database, and synchronisation safety errors still fail the run.

## Decisions and trade-offs

### Keep source order explicit

I kept Icecast as the preferred source because it provides structured status data. The player page is a fallback for source failure or missing metadata, not the new default. Retrying Icecast on the next poll lets the structured source recover without a manual configuration change.

### Record only what a poll can show

Each HTTP attempt gets a unique query value and no-cache/no-store headers, so the client never reuses a cached response. The broadcaster can still serve an older page, so a page may show a stale track. The stored poll time therefore means **observed at**, not the time a track first played or was published.

### Isolate optional-source failure

`run-once` handles the specific case where both radio sources are unavailable and proceeds without a new radio observation. Every other safety error still stops the workflow. A radio metadata outage cannot turn into a database, file, or synchronisation error, and the logger never invents or replays an observation to fill the gap.

The cost is an HTML parsing path and a second source format to maintain. I accepted that to keep logging continuous when the structured endpoint is down.

## Where to look

The [README polling contract](https://github.com/wouthh/hardcore-radio-logger/blob/main/README.md#using-the-built-in-poller) describes the same boundary. The [poller](https://github.com/wouthh/hardcore-radio-logger/blob/main/hcr_sync/poller.py) holds the source ordering and page parsing, and the [`run-once` command](https://github.com/wouthh/hardcore-radio-logger/blob/main/hcr_sync/cli.py) handles the unavailable-source case. Synthetic fallback tests and a workflow-continuation test live in [the poller tests](https://github.com/wouthh/hardcore-radio-logger/blob/main/tests/test_poller_fallback.py) and [the CLI tests](https://github.com/wouthh/hardcore-radio-logger/blob/main/tests/test_cli.py).
