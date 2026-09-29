# How it works

The chart runs three pieces around one image:

- A Kubernetes `CronJob` that starts `drumsergio/duplicacy-container` as a one-shot Job on your schedule
  and runs `run-parts /etc/periodic/daily`, so every wrapper script in that directory runs once.
- A `duplicacy-exporter` Deployment that turns backup activity into Prometheus metrics. In `log_tail`
  mode it tails the log file the Job writes to a shared PVC; in `webhook` mode it receives Duplicacy
  Web UI reports instead.
- An optional Web UI Deployment, disabled by default.

## Why this repo exists

The Duplicacy Kubernetes story spans multiple sibling projects:

- `duplicacy-cli-cron` handles the backup execution model
- `duplicacy-exporter` turns backup activity into Prometheus metrics
- the chart glues them together into a reusable stack with PVCs, secrets, ingress, and monitoring

That stack is bigger than an exporter-only concern, so it lives better in its own repo.

## What is in the repo

- `Dockerfile` + `entrypoint.sh` - runtime image for one-shot or cron-style Duplicacy execution
- `charts/duplicacy` - Duplicacy stack chart with backup job, exporter, and optional Web UI

## The runtime image

The bundled image is intentionally minimal:

- it ships Duplicacy CLI 3.2.5 plus `shoutrrr` 0.8.0 on Alpine 3.23
- it keeps the familiar periodic directory layout
- it expects operators to mount their own `/config` and `/etc/periodic/*` content

That lets `duplicacy-cli-cron` stay focused on reusable scripts and backup recipes while this repo owns the runtime/container distribution.

## The Web UI

The Web UI component is intentionally:

- `disabled` by default
- generic rather than hardcoded to one wrapper image
- configured through `web.image.*`, `web.command`, `web.args`, and optional ingress/PVC settings

This keeps the chart useful without baking in a deployment choice that may not fit every operator or licensing situation.
