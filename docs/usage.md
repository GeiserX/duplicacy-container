# Usage

## Backups

The CronJob starts a one-shot Job on `cron.schedule`. The Job runs `run-parts /etc/periodic/daily`, so
every executable script in that directory runs once, in name order. With `sharedLogs` on, the output is
also appended to `cron.logFile` (`/logs/duplicacy.log`) on the shared PVC.

```bash
kubectl get cronjob,jobs -l app.kubernetes.io/component=cron
kubectl create job --from=cronjob/duplicacy-cron duplicacy-manual   # run once now
```

`concurrencyPolicy: Forbid` keeps a second run from starting while one is still going.

## Metrics

The exporter Deployment serves `/metrics` and `/health` on port 9750. In the default `log_tail` mode it
reads the Job's log file from the shared PVC; set `exporter.mode=webhook` to receive Duplicacy Web UI
reports on `/webhook` instead. Set `exporter.serviceMonitor.enabled=true` if you run the Prometheus
Operator. The metrics themselves are documented in
[duplicacy-exporter](https://github.com/GeiserX/duplicacy-exporter/blob/main/docs/metrics.md).

## Web UI

Off by default. Set `web.enabled=true` with `web.image.repository` and `web.image.tag` for the image you
trust; it listens on port 3875, keeps `/config` on a PVC, and can get an Ingress through `web.ingress.*`.

```bash
kubectl port-forward svc/duplicacy-web 3875:3875
```
