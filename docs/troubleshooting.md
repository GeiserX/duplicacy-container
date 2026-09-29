# Troubleshooting

## The Job runs but backs up nothing

`run-parts` found no scripts. Check that `cron.periodic` mounts a PVC or ConfigMap with your wrapper
scripts at `/etc/periodic/daily`, and that `cron.config` mounts `/config`; the install notes warn when
either mount is missing. Scripts from a ConfigMap need names `run-parts` accepts: letters, digits,
underscores and hyphens, no dots.

## `helm install` fails with "exporter.mode=log_tail requires sharedLogs.enabled=true"

The exporter tails the Job's log file from the shared PVC. Turn `sharedLogs` back on, or set
`exporter.mode=webhook`.

## `helm install` fails with "Set web.image.repository when web.enabled=true"

The Web UI has no default image on purpose. Set `web.image.repository` and `web.image.tag`.

## The exporter shows no backups

In `log_tail` mode the exporter only sees what the Job writes to `cron.logFile`. Check that the shared
log PVC is bound in both pods (`kubectl get pvc`); with a `ReadWriteOnce` class the two pods must land on
the same node. The Job's log lines need a snapshot ID and machine name for the exporter to label them;
see [duplicacy-exporter's troubleshooting](https://github.com/GeiserX/duplicacy-exporter/blob/main/docs/troubleshooting.md).

## Reporting a bug

Open an issue at https://github.com/GeiserX/duplicacy-container/issues with the chart version, your
values with credentials removed, `kubectl describe` of the failing pod, and its logs.
