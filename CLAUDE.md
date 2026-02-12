# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of independent Google Cloud Functions examples (Node.js) from the [Google Cloud Functions Tutorial Series](https://rominirani.com/google-cloud-functions-tutorial-series-f04b2db739cd). Each subdirectory is a self-contained, independently deployable function with its own `package.json`.

## Deployment

Functions are deployed individually via `gcloud` CLI — there is no shared build system, monorepo tooling, or test framework.

```bash
# HTTP-triggered function
gcloud functions deploy <exportedFunctionName> --trigger-http --region=us-central1 --runtime=nodejs6

# Cloud Storage-triggered function
gcloud functions deploy <exportedFunctionName> --trigger-resource gs://<bucket> \
  --trigger-event google.storage.object.finalize --region=us-central1 --runtime=nodejs6

# Pub/Sub-triggered function
gcloud functions deploy <exportedFunctionName> --trigger-resource <topic> \
  --trigger-event google.pubsub.topic.publish --region=us-central1 --runtime=nodejs6
```

Install dependencies per-function: `cd <function-dir> && npm install`

See `deploy.txt` for all deployment commands and `cleanup.txt` for teardown.

## Useful gcloud Commands

```bash
gcloud functions list
gcloud functions describe <name>
gcloud functions call <name> --data '{"key":"value"}'
gcloud functions logs read --limit=10
```

## Architecture

Each function exports a single named handler from `index.js`. Two patterns:

- **HTTP triggers:** `exports.fn = (req, res) => { ... }` — Express-like request/response
- **Event triggers (GCS/Pub/Sub):** `exports.fn = (event, callback) => { ... }` — event data in `event.data`, must call `callback()`

## Function Index

| Directory | Trigger | What it demonstrates |
|---|---|---|
| `helloworld-http` | HTTP | Minimal hello world |
| `hellogreeting-http` | HTTP | JSON body parameter handling |
| `uuid-http` | HTTP | npm dependency usage (`uuid`) |
| `hello-gcs` | GCS | Cloud Storage event handling |
| `hello-pubsub` | Pub/Sub | Message subscription, base64 decoding |
| `hello-localfunctions` | HTTP | Local emulator debugging |
| `bitcoinprice-http` | HTTP | External API calls (Axios), Hangouts Chat |
| `getrandomquote-function` | HTTP | Simple JSON API endpoint |
| `firestore-http` | HTTP | Firestore queries via Firebase Admin SDK |
| `translation-process` | GCS | Cloud Translation API, stream-based I/O |
| `puppeteer` | HTTP | Headless Chrome scraping (needs `--memory=1024MB`) |
| `environment` | HTTP | Environment variable access |

## Runtime Notes

- Most functions target Node.js 6; `puppeteer` requires Node.js 8
- The `puppeteer` function requires `--memory=1024MB` at deploy time
- Pub/Sub and GCS triggers require creating the topic/bucket first (see `deploy.txt`)
