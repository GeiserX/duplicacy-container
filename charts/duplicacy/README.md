# Duplicacy Helm Chart

This chart packages the full Duplicacy stack for Kubernetes:

- A Kubernetes `CronJob` that runs `run-parts /etc/periodic/daily` once per schedule using `drumsergio/duplicacy-container`
- An optional `duplicacy-exporter` Deployment that exposes Prometheus metrics and can tail the same log file
- An optional, generic Web UI pod so operators can bring their preferred Duplicacy UI wrapper if they want one

## Design Notes

- The backup image is run as a one-shot Kubernetes Job. The chart does not daemonize `crond` inside the backup pod.
- The exporter defaults to `MODE=log_tail` and reads `cron.logFile`.
- The Web UI is disabled by default and intentionally generic so the chart does not hardcode an unofficial UI image.
- You can provide Duplicacy scripts from either existing PVCs or ConfigMaps mounted at `/config` and `/etc/periodic`.
- `sharedLogs` assumes an RWX-capable storage class by default because the CronJob pod and exporter may run on different nodes.

## Before Installing

You still need Duplicacy repository configuration and daily wrapper scripts. The chart does not generate those for you.

Typical inputs:

- `/config`: `dual-executor.sh`, `config-s3.sh`, and any supporting scripts
- `/etc/periodic/daily`: one wrapper script per backup target

## Minimal Example

```yaml
cron:
  schedule: "0 2 * * *"
  config:
    configMapName: duplicacy-config
  periodic:
    configMapName: duplicacy-periodic
  storage:
    endpoint1: garage-a:9000
    endpoint2: garage-b:9000
    bucket: duplicacy
    region: garage
  credentials:
    APPDATA:
      s3Id: primary-key
      s3Secret: primary-secret
      password: repo-password
    APPDATAC:
      s3Id: secondary-key
      s3Secret: secondary-secret
      password: repo-password

sharedLogs:
  enabled: true

exporter:
  enabled: true
  serviceMonitor:
    enabled: true
```

Install with:

```bash
helm upgrade --install duplicacy ./charts/duplicacy -f my-values.yaml
```

## Optional Web UI Example

```yaml
web:
  enabled: true
  image:
    repository: saspus/duplicacy-web
    tag: 1.0.0
  ingress:
    enabled: true
    hosts:
      - host: duplicacy.example.com
        paths:
          - path: /
            pathType: Prefix
```

The Web UI block stays generic on purpose:

- Set `web.image.*` yourself when enabling it
- Mounts `/config` on a PVC and gives `/logs` and `/cache` ephemeral storage by default
- Lets you pass extra env, envFrom, volumes, mounts, ingress, and resource settings

## Useful Values

- `cron.config.*`: mount `/config` from a PVC or ConfigMap
- `cron.periodic.*`: mount daily wrapper scripts, typically at `/etc/periodic/daily`
- The default `cron.periodic.mountPath` is `/etc/periodic/daily`, which makes ConfigMap-based wrapper scripts work out of the box
- `cron.existingSecret`: reuse a pre-created Secret instead of rendering one from values
- `cron.extraEnvFrom`: bring in External Secrets or other ConfigMaps
- `cron.timeZone`: pin the CronJob to an explicit timezone instead of relying on cluster defaults
- `sharedLogs.existingClaim`: use an existing PVC for `LOG_FILE` mode
- `sharedLogs.accessModes`: keep `ReadWriteMany` unless you are intentionally constraining both workloads to the same node
- `exporter.storageHostMap`: map raw storage hosts/IPs to friendly labels in metrics
- `exporter.tailscaleDomain`: optional friendly domain suffix for exporter labels; left empty by default
- `web.persistence.config.*`: keep Web UI config on a PVC while leaving logs/cache ephemeral unless you override them

## Monitoring

- Metrics endpoint: `/metrics`
- Health endpoint: `/health`
- Optional Prometheus Operator integration via `exporter.serviceMonitor.enabled`
