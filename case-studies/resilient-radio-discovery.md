# Resilient Radio Discovery

How a public local-first tool separates optional radio-source outages from local workflow failures.

- **Public source:** [Hardcore Radio Logger](https://github.com/wouthh/hardcore-radio-logger), inspected at `main` commit `225b53bd2f4409759ce8089f91aa2f08eabe87cc`, the merge commit for [PR 6](https://github.com/wouthh/hardcore-radio-logger/pull/6).
- **Evidence boundary:** This page describes repository behaviour at that commit. It does not establish who authored, reviewed, or tested the change, or whether it ran in a deployed environment.

## Summary

The poller asks the configured Icecast status endpoint for track metadata first. When Icecast is unavailable or returns no usable track, it tries the broadcaster's official player page. Each new poll starts with Icecast again, so the fallback does not become a permanent source preference.

The useful boundary is what happens when neither source has usable metadata. A standalone `poll-radio` call fails explicitly. The larger `run-once` workflow records a warning and continues its later stages without adding a radio observation. In verbose apply mode it also records an unavailable-poll event with sanitised failure reasons. Local file, logger, database, and synchronisation safety errors still fail the run.

## Decisions and trade-offs

### Keep source order explicit

Icecast remains the preferred source because it provides structured status data. The player page is a fallback for source failure or missing metadata, not the new default. Retrying Icecast on the next poll lets the structured source recover without a manual configuration change.

### Preserve what the poll can establish

The poller adds a unique query value and no-cache/no-store headers to each HTTP attempt. This avoids reusing a client-cached response, but it cannot force the broadcaster to serve fresh page data. A page may still show an older track. The stored poll time means **observed at**; it does not establish when a track first played or was published.

### Isolate optional-source failure

`run-once` handles the specific case where both radio sources are unavailable, then proceeds without a new radio observation. Other safety errors continue to stop the workflow. This keeps a radio metadata outage from silently becoming a database, file, or synchronisation error, while avoiding an invented or replayed observation.

The fallback also adds an HTML parsing path and another source format to maintain. That is the cost of improving continuity when the structured endpoint is down; it does not guarantee that the webpage remains available or current.

## Evidence and limits

The [README polling contract](https://github.com/wouthh/hardcore-radio-logger/blob/225b53bd2f4409759ce8089f91aa2f08eabe87cc/README.md#using-the-built-in-poller) describes the same boundary. The [poller implementation](https://github.com/wouthh/hardcore-radio-logger/blob/225b53bd2f4409759ce8089f91aa2f08eabe87cc/hcr_sync/poller.py) contains the source ordering and page parsing, while the [`run-once` boundary](https://github.com/wouthh/hardcore-radio-logger/blob/225b53bd2f4409759ce8089f91aa2f08eabe87cc/hcr_sync/cli.py) handles the unavailable-source case. The merged change adds synthetic fallback tests and a workflow-continuation test in [the poller tests](https://github.com/wouthh/hardcore-radio-logger/blob/225b53bd2f4409759ce8089f91aa2f08eabe87cc/tests/test_poller_fallback.py) and [the CLI tests](https://github.com/wouthh/hardcore-radio-logger/blob/225b53bd2f4409759ce8089f91aa2f08eabe87cc/tests/test_cli.py). Their presence does not mean they were run during this portfolio update.

This source snapshot supports claims about code and tests present at one public commit. It does not prove human review, live broadcaster behaviour, runtime deployment, provider availability, or production uptime.
