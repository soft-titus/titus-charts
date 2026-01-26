# Job

This reusable Helm chart deploys Kubernetes Job and CronJob resources
using a single shared jobTemplate. It is designed for batch workloads,
scheduled tasks, and one-off jobs with GitOps-friendly behavior and
safe defaults.

The chart can create:
- A standalone Job
- A CronJob
- Both Job and CronJob sharing the same configuration

Container image automation is supported via Flux Image Automation
or Argo CD Image Updater, enabling declarative image updates through
GitOps workflows.

----------------------------------------------------------------------
## Prerequisites

- Kubernetes 1.23+
- Helm 3.x
- Flux 2.7.0+ (only if jobTemplate.image.fluxImageAutoUpdate is enabled)
  - image-reflector-controller
  - image-automation-controller
- ArgoCD + ArgoCD Image Updater (only if jobTemplate.image.argocdImageUpdater is enabled)

----------------------------------------------------------------------
## Usage

Add the chart repository:

```bash
helm repo add titus-charts http://soft-titus.github.io/titus-charts
helm repo update
```

Install the chart:

```bash
helm install demo titus-charts/job
```

Uninstall the chart:

```bash
helm delete demo
```

----------------------------------------------------------------------
## Behavior Overview

- enabled=false disables rendering of ALL resources (GitOps-friendly)
- jobTemplate defines the shared Job spec for Job and CronJob
- job.enabled controls standalone Job creation
- cronJob.enabled controls CronJob creation
- Sidecars and initContainers are supported
- Sidecars must terminate for Jobs to complete

----------------------------------------------------------------------
## Values Reference

For detailed examples and full documentation, see values.yaml.

| Key | Type | Default | Description / Notes |
|-----|------|---------|---------------------|
| enabled | bool | true | Master switch to render or disable all resources |
| nameOverride | string | "" | Override chart name |
| fullnameOverride | string | "" | Fully override resource names |
| job.enabled | bool | true | Create a standalone Job |
| cronJob.enabled | bool | true | Create a CronJob |
| cronJob.schedule | string | "0 0 * * *" | Cron schedule |
| cronJob.suspend | bool | false | Suspend future executions |
| cronJob.concurrencyPolicy | string | Allow | Allow, Forbid, Replace |
| cronJob.failedJobsHistoryLimit | int | 1 | Failed jobs to retain |
| cronJob.successfulJobsHistoryLimit | int | 3 | Successful jobs to retain |
| jobTemplate.image.repository | string | busybox | Container image repository |
| jobTemplate.image.tag | string | "1.37.0" | Container image tag |
| jobTemplate.image.pullPolicy | string | IfNotPresent | Image pull policy |
| jobTemplate.image.fluxImageAutoUpdate.enabled	| bool	| false	| Enable Flux-based image auto-update (creates ImageRepository and ImagePolicy) |
| jobTemplate.image.fluxImageAutoUpdate.namespace | string | flux-system | Namespace where Flux ImageRepository and ImagePolicy are created. |
| jobTemplate.image.fluxImageAutoUpdate.pattern	| string	| "^[1-9]+\\.[0-9]+\\.[0-9]+$"	| Regex pattern used to filter image tags (must match SemVer if semver is set) |
| jobTemplate.image.fluxImageAutoUpdate.semver	| string	| ">=1.0.0 <2.0.0"	| SemVer range used by Flux ImagePolicy to select the latest image |
| jobTemplate.image.argocdImageUpdater.enabled	| bool	| false	| Enable Argo CD Image Updater support |
| jobTemplate.image.argocdImageUpdater.applicationNamespace	| string	| argocd	| Namespace where Argo CD Applications are deployed |
| jobTemplate.image.argocdImageUpdater.applicationNamePattern	| string	| ""	| Target Application name or glob pattern (defaults to release fullname) |
| jobTemplate.image.argocdImageUpdater.images	| list	| []	| List of images managed by Argo CD Image Updater |
| jobTemplate.image.argocdImageUpdater.writeBackConfig.method	| string	| argocd	| Write-back method: argocd or git |
| jobTemplate.image.argocdImageUpdater.writeBackConfig.gitConfig	| map	| {}	| Git write-back configuration |
| jobTemplate.imagePullSecrets | list | [] | Image pull secrets |
| jobTemplate.command | list | [] | Override container entrypoint |
| jobTemplate.args | list | [] | Override container arguments |
| jobTemplate.env | list | [] | Environment variables |
| jobTemplate.envFrom | list | [] | Import env vars from ConfigMaps or Secrets |
| jobTemplate.configMaps.env.enabled | bool | false | Enable env ConfigMap |
| jobTemplate.configMaps.env.name | string | "" | Name of env ConfigMap (auto-generated if empty) |
| jobTemplate.configMaps.env.data | map | {} | Key/value env variables |
| jobTemplate.configMaps.files.enabled | bool | false | Enable files ConfigMap |
| jobTemplate.configMaps.files.name | string | "" | Name of files ConfigMap |
| jobTemplate.configMaps.files.mountPath | string | /app/config | Mount path for files ConfigMap |
| jobTemplate.configMaps.files.data | map | {} | File contents (text) |
| jobTemplate.configMaps.binaryFiles.enabled | bool | false | Enable binary files ConfigMap |
| jobTemplate.configMaps.binaryFiles.name | string | "" | Name of binary files ConfigMap |
| jobTemplate.configMaps.binaryFiles.mountPath | string | /app/binary | Mount path for binary files |
| jobTemplate.configMaps.binaryFiles.data | map | {} | Base64-encoded binary file data |
| jobTemplate.resources.requests.cpu | string | "100m" | CPU request |
| jobTemplate.resources.requests.memory | string | "128Mi" | Memory request |
| jobTemplate.resources.limits.cpu | string | "500m" | CPU limit |
| jobTemplate.resources.limits.memory | string | "512Mi" | Memory limit |
| jobTemplate.serviceAccount.create | bool | false | Create ServiceAccount |
| jobTemplate.serviceAccount.automount | bool | false | Automount SA token |
| jobTemplate.serviceAccount.name | string | "" | ServiceAccount name |
| jobTemplate.podAnnotations | map | {} | Pod annotations |
| jobTemplate.podLabels | map | {} | Pod labels |
| jobTemplate.podSecurityContext | object | {} | Pod security context |
| jobTemplate.securityContext | object | {} | Container security context |
| jobTemplate.volumes | list | [] | Additional volumes |
| jobTemplate.volumeMounts | list | [] | Volume mounts |
| jobTemplate.nodeSelector | map | {} | Node selector |
| jobTemplate.tolerations | list | [] | Pod tolerations |
| jobTemplate.affinity | object | {} | Affinity rules |
| jobTemplate.priorityClassName | string | "" | PriorityClass name |
| jobTemplate.restartPolicy | string | OnFailure | Restart policy |
| jobTemplate.initContainers | list | [] | Init containers |
| jobTemplate.sidecars | list | [] | Sidecar containers |
