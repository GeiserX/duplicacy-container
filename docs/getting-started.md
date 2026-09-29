# Getting started

## What you need first

- A Kubernetes cluster (1.21 or later) and Helm.
- Your Duplicacy scripts: `/config` with `dual-executor.sh`, `config-s3.sh` and anything they call, and
  `/etc/periodic/daily` with one wrapper script per backup target. The chart mounts them from a PVC or a
  ConfigMap; it does not generate them. [duplicacy-cli-cron](https://github.com/GeiserX/duplicacy-cli-cron)
  has the scripts and recipes.
- Storage credentials and the repository encryption password, either as values or in a Secret you
  already have.
- For the exporter's default `log_tail` mode, a storage class that can do `ReadWriteMany`, because the
  backup Job and the exporter share the log PVC and may run on different nodes.

## Install

Write a `my-values.yaml`. The smallest useful one mounts your scripts and sets the storage and
credentials:

```yaml
cron:
  schedule: "0 2 * * *"
  config:
    configMapName: duplicacy-config
  periodic:
    configMapName: duplicacy-periodic
  storage:
    endpoint: s3.example.com:9000
    bucket: duplicacy
    region: us-east-1
  credentials:
    APPDATA:
      s3Id: access-key
      s3Secret: secret-key
      password: repo-password
```

To keep credentials out of values, create a Secret with the same keys the chart would render
(`ENDPOINT_1`, `BUCKET`, `REGION`, `DUPLICACY_<NAME>_S3_ID`, `DUPLICACY_<NAME>_S3_SECRET`,
`DUPLICACY_<NAME>_PASSWORD`, optionally `SHOUTRRR_URL`) and set `cron.existingSecret` to its name.

```bash
helm repo add duplicacy https://geiserx.github.io/duplicacy-container
helm install duplicacy duplicacy/duplicacy -f my-values.yaml
```

Every value is in [Configuration](configuration.md).

## What working looks like

The install notes print these commands. After the first scheduled run:

```bash
kubectl get jobs -l app.kubernetes.io/component=cron
kubectl logs job/<the newest job name>
```

The Job shows `1/1` completions and its log ends with your wrapper scripts' Duplicacy output. To try a
run without waiting for the schedule, start one from the CronJob:

```bash
kubectl create job --from=cronjob/duplicacy-cron duplicacy-manual
```

The exporter answers on port 9750:

```bash
kubectl port-forward svc/duplicacy-exporter 9750:9750
curl http://localhost:9750/metrics
```
