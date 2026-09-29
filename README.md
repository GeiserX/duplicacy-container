<p align="center">
  <img src="https://raw.githubusercontent.com/GeiserX/duplicacy-container/main/docs/images/banner.svg" alt="duplicacy-container" width="900"/>
</p>

# duplicacy-container

<p>
  <a href="https://github.com/GeiserX/duplicacy-container/releases"><img src="https://img.shields.io/github/v/release/GeiserX/duplicacy-container?style=flat-square" alt="Release"></a>
  <a href="https://github.com/GeiserX/duplicacy-container/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/GeiserX/duplicacy-container/ci.yml?style=flat-square&label=CI" alt="CI"></a>
  <a href="https://github.com/GeiserX/duplicacy-container/blob/main/LICENSE"><img src="https://img.shields.io/github/license/GeiserX/duplicacy-container?style=flat-square" alt="License"></a>
  <a href="https://artifacthub.io/packages/helm/duplicacy/duplicacy"><img src="https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/duplicacy&style=flat-square" alt="ArtifactHub"></a>
</p>

Container image and Helm chart for running a Duplicacy backup stack on Kubernetes: a scheduled backup Job, the [duplicacy-exporter](https://github.com/GeiserX/duplicacy-exporter) Deployment for Prometheus metrics, and an optional Web UI pod.

## Features

- The `drumsergio/duplicacy-container` runtime image: Duplicacy CLI 3.2.5 and shoutrrr, kept apart from the scripts repo.
- Backups run as one-shot Kubernetes Jobs from a CronJob instead of a long-running cron container.
- Shared log PVC so `duplicacy-exporter` can tail the backup output in `log_tail` mode.
- Optional webhook mode for the Duplicacy Web UI.
- Optional, generic Web UI pod on port `3875`, disabled by default.
- `values.schema.json`, a chart README with the values reference, and Helm CI and release workflows.
- Existing Secrets and PVCs, a ServiceMonitor, and `extraEnv`/`extraVolumes` escape hatches.

## Quick start

```bash
helm repo add duplicacy https://geiserx.github.io/duplicacy-container
helm install duplicacy duplicacy/duplicacy -f my-values.yaml
```

`my-values.yaml` mounts your Duplicacy scripts (`cron.config`, `cron.periodic`) and sets the storage and credentials (`cron.storage` and `cron.credentials`, or `cron.existingSecret`); [Getting started](https://github.com/GeiserX/duplicacy-container/blob/main/docs/getting-started.md) has a minimal one, and every key is in the [chart README](https://github.com/GeiserX/duplicacy-container/blob/main/charts/duplicacy/README.md).

## Documentation

- [Getting started](https://github.com/GeiserX/duplicacy-container/blob/main/docs/getting-started.md): what you need first, a minimal values file, install, what a first run looks like
- [Configuration](https://github.com/GeiserX/duplicacy-container/blob/main/docs/configuration.md): the values that matter, and where the full reference lives
- [Usage](https://github.com/GeiserX/duplicacy-container/blob/main/docs/usage.md): backups, metrics, the Web UI
- [How it works](https://github.com/GeiserX/duplicacy-container/blob/main/docs/how-it-works.md): the Job, the exporter, the runtime image, why this repo exists
- [Troubleshooting](https://github.com/GeiserX/duplicacy-container/blob/main/docs/troubleshooting.md)

## Related projects

[duplicacy-cli-cron](https://github.com/GeiserX/duplicacy-cli-cron) (the scripts), [duplicacy-exporter](https://github.com/GeiserX/duplicacy-exporter), [duplicacy-ha](https://github.com/GeiserX/duplicacy-ha), [duplicacy-mcp](https://github.com/GeiserX/duplicacy-mcp), and [Duplicacy](https://duplicacy.com) itself.

## License

[GPL-3.0-or-later](https://github.com/GeiserX/duplicacy-container/blob/main/LICENSE)
