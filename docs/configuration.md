# Configuration

The full values reference, with a minimal example and the optional Web UI example, is the chart README:
[charts/duplicacy/README.md](https://github.com/GeiserX/duplicacy-container/blob/main/charts/duplicacy/README.md). It lives there because ArtifactHub shows it on
the chart's page. The defaults are in [values.yaml](https://github.com/GeiserX/duplicacy-container/blob/main/charts/duplicacy/values.yaml), and
[values.schema.json](https://github.com/GeiserX/duplicacy-container/blob/main/charts/duplicacy/values.schema.json) validates what you pass.

The values most installs touch:

| Value | Default | What it does |
|---|---|---|
| `cron.schedule` | `0 2 * * *` | When the backup Job runs |
| `cron.config.*`, `cron.periodic.*` | none | Where `/config` and `/etc/periodic/daily` come from: `existingClaim` or `configMapName` |
| `cron.storage.*`, `cron.credentials` | empty | Storage endpoint, bucket, region and per-repository credentials, rendered into a Secret |
| `cron.existingSecret` | `""` | Use your own Secret instead of the rendered one |
| `cron.notification.shoutrrrUrl` | `""` | Where the scripts send notifications |
| `sharedLogs.enabled` | `true` | The log PVC the exporter tails in `log_tail` mode |
| `exporter.mode` | `log_tail` | `log_tail` or `webhook` |
| `exporter.serviceMonitor.enabled` | `false` | Create a Prometheus Operator ServiceMonitor |
| `web.enabled` | `false` | Deploy a Web UI pod; needs `web.image.repository` and `web.image.tag` |
