# Batch Processing on the JVM

Professional batch work on Java, Kotlin, and Spring Batch, with a React and Electron desktop client from the same assignment.

- **Stack:** Java, Kotlin, Spring Batch, React, Electron, SQL Server, PostgreSQL

## Context

The team handed me the processing work that had to be automated, and I built the batch jobs that carried it out. Each job had to give the same result on test data and on real data, and it had to recover cleanly when a run stopped partway through.

## What I did

- I designed and built the batch processes from the requirements I was given.
- I built their failure and restart handling: chunk transactions, skip and retry rules, restartable jobs, idempotent writes, and reprocessing.
- I supervised the jobs in operation and checked their output on both test data and real data.
- I argued for and built the Electron client that automated the software the team worked with.

## Decisions

### Restartable chunks with idempotent writes

I processed records in chunks, each committed in its own transaction, so a failure rolls back only the chunk in progress. Jobs are restartable from their last committed chunk, and writes are idempotent, so a restart or a deliberate reprocessing run does not duplicate results. Skip and retry rules separate records that can be retried from records that should be set aside, so one bad record does not stop the whole run.

### Electron for desktop automation

The project had an ambitious goal: the program had to open and drive the other software the team used, while that software gave no control over what opens or how. I argued for Electron because it behaves like a browser and can be scripted. With Electron and custom JavaScript that runs inside the opened pages, I made that automation possible.

## How it was verified

I ran each job against test data first and then against real data, and supervised the runs to confirm their output before relying on them.
