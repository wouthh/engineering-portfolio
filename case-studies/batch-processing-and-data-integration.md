# Batch Processing and Data Integration

Batch processing of insurance contracts and claims on Java, Kotlin, and Spring Batch, a React workstation running inside Electron that automated a claims web application, and file-based integration work in PHP.

- **Stack:** Java, Kotlin, Spring Batch, React, Electron, PHP, CSV/XML/JSON, file transfer.

## Context

The batch jobs processed insurance contracts and claims. The team handed me the processing work that had to be automated, and I built the jobs that carried it out. Each job had to apply the same processing rules to test data and to real data and produce correct output for each, and it had to recover cleanly when a run stopped partway through.

## What I did

- I designed and built the batch processes from the requirements I was given.
- I built their failure and restart handling: chunk transactions, skipping and logging failing records, restartable jobs, duplicate protection, and reprocessing.
- I supervised the jobs in operation and checked their output on both test data and real data.
- I argued for Electron and built the automation in the claims-handling workstation, a React application running inside Electron, so a handler could open a contract or claim in an insurer's claims web application without clicking through it.

## Decisions

### Duplicate protection lives in the database

A rerun must never create a second copy of work that already happened. I handled that in the database, because that was the preferred place for it, and I combined three mechanisms that I worked with often:

- **Processed states** on the input records mark which records a run has handled, so a restart picks up only the rest.
- **Unique constraints** on the target data make a duplicate impossible, whichever job or person writes.
- **Upserts** on a business key turn a rerun into an update instead of a second row.

Putting the guarantee in the database means it holds for every writer, not only for code that remembers to check. It has costs: an upsert depends on a reliable business key, the processed state has to change in the same transaction as the write, and constraint violations need clear handling so they read as "already done" rather than as errors. Chunk commits support this: each chunk commits on its own, so a restart resumes after the last committed chunk.

### Skip and log, never stop the run

A failing record is skipped and logged, so one bad record cannot block a whole batch. The trade-off is that the log becomes part of the process, and skipped records need follow-up.

### A workstation that drives a web application

Claim handlers used an insurer-specific claims web application that needed many repetitive manual clicks, and it offered no integration to drive it from outside. In the workstation, a handler now searches for a contract or claim and clicks "open". The workstation opens the insurer's web application, types the reference into its search bar, and opens the matching claim, so the handler lands directly on it.

I argued for Electron because it embeds a browser engine and lets scripts run inside the pages it opens. The web application had no other way to be driven.

## File-based imports and exports

On another assignment I wrote file-based imports and exports of various data for click-and-collect shops in PHP. The work covered CSV, XML, and JSON processing and the file-transfer workflows that moved those files between systems.

## How it was verified

I ran each job against test data first and then against real data, and supervised the runs to confirm their output before relying on them.
