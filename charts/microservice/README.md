# Microservice (Helm Chart)

This reusable Helm chart can be used to deploy a generic microservice
on Kubernetes. It is designed to provide flexible configuration,
GitOps-friendly behavior, and safe defaults.

The chart deploys a Deployment and can optionally create ConfigMaps,
ServiceAccounts, Services, Ingresses, HTTPRoutes, PVCs, HPAs, VPAs, and
PDBs. It also supports Flux ImageRepository and ImagePolicy resources
for automated image updates. 

----------------------------------------------------------------------
## Prerequisites

- Kubernetes 1.23+
- Helm 3.x
- Ingress controller (only if ingress is enabled)
- Gateway API installed (only if httpRoute.enabled=true)
- Metrics Server (only if HPA or VPA is enabled)
- VPA (only if VPA is enabled)
- Flux 2.7.0+ (only if image.fluxImageAutoUpdate is enabled)
  with the following components:
  - image-reflector-controller
  - image-automation-controller

----------------------------------------------------------------------
## Usage

Add the chart repository:

``` bash
helm repo add titus-charts http://soft-titus.github.io/titus-charts
helm repo update
```

To install the chart:

``` bash
helm install demo titus-charts/microservice
```

To uninstall the chart:

``` bash
helm delete demo
```

----------------------------------------------------------------------
## Values Reference

The table below lists all supported values for this chart, their types, defaults, and a brief description. For detailed examples, refer to `values.yaml`.

| Key | Type | Default | Description / Notes |
|-----|------|---------|---------------------|
| enabled | bool | true | Enable or disable rendering of all resources (GitOps-friendly switch) |
| nameOverride | string | "" | Override chart name used in templates |
| fullnameOverride | string | "" | Fully override generated resource names |
| image.repository | string | ghcr.io/soft-titus/hello | Container image repository |
| image.tag | string | "1.0.0" | Container image tag |
| image.pullPolicy | string | IfNotPresent | Image pull policy: Always, IfNotPresent, Never |
| image.fluxImageAutoUpdate.enabled	| bool	| false	| Enable Flux-based image auto-update (creates ImageRepository and ImagePolicy) |
| image.fluxImageAutoUpdate.pattern	| string	| "^[1-9]+\\.[0-9]+\\.[0-9]+$"	| Regex pattern used to filter image tags (must match SemVer if semver is set) |
| image.fluxImageAutoUpdate.semver	| string	| ">=1.0.0 <2.0.0"	| SemVer range used by Flux ImagePolicy to select the latest image |
| imagePullSecrets | list | [] | Secrets for pulling images from private registries |
| command | list | [] | Override container entrypoint |
| args | list | [] | Override container command arguments |
| env | list | [] | Environment variables (supports value / valueFrom) |
| envFrom | list | [] | Import env vars from ConfigMaps or Secrets |
| configMaps.env.enabled | bool | false | Enable env ConfigMap |
| configMaps.env.name | string | "" | Name of env ConfigMap (auto-generated if empty) |
| configMaps.env.data | map | {} | Key/value env variables |
| configMaps.files.enabled | bool | false | Enable files ConfigMap |
| configMaps.files.name | string | "" | Name of files ConfigMap |
| configMaps.files.mountPath | string | /app/config | Mount path for files ConfigMap |
| configMaps.files.data | map | {} | File contents (text) |
| configMaps.binaryFiles.enabled | bool | false | Enable binary files ConfigMap |
| configMaps.binaryFiles.name | string | "" | Name of binary files ConfigMap |
| configMaps.binaryFiles.mountPath | string | /app/binary | Mount path for binary files |
| configMaps.binaryFiles.data | map | {} | Base64-encoded binary file data |
| pvc.enabled | bool | false | Enable PersistentVolumeClaim |
| pvc.name | string | "" | PVC name (auto-generated if empty) |
| pvc.storageClassName | string | "" | StorageClass name |
| pvc.accessModes | list | [ReadWriteOnce] | PVC access modes |
| pvc.resources.requests.storage | string | 1Gi | Storage size request |
| pvc.mountPath | string | /data | Volume mount path in container |
| livenessProbe | object | {} | Liveness probe configuration |
| readinessProbe | object | {} | Readiness probe configuration |
| resources.requests.cpu | string | "100m" | CPU request |
| resources.requests.memory | string | "128Mi" | Memory request |
| resources.limits.cpu | string | "500m" | CPU limit |
| resources.limits.memory | string | "512Mi" | Memory limit |
| vpa.enabled	| bool	| false	| Enable or disable Vertical Pod Autoscaler (VPA). Cannot be used with HPA on the same resource for CPU/memory. |
| vpa.updateMode	| string	| Off	| Update mode: Off, Initial, Recreate, or InPlaceOrRecreate. Determines how recommendations are applied. |
| vpa.minReplicas	| int	| null	| Optional safeguard: VPA will not evict pods if replicas are below this number. |
| vpa.resourcePolicy.containerPolicies	| list	| []	| Fine-grained resource control per container. Example: min/max allowed resources, controlled resource types. |
| replicaCount | int | 1 | Number of pod replicas |
| deploymentStrategy.type | string | RollingUpdate | Deployment strategy (RollingUpdate, Recreate) |
| deploymentStrategy.rollingUpdate.maxUnavailable | string/int | 0 | Max unavailable pods during update |
| deploymentStrategy.rollingUpdate.maxSurge | string/int | 1 | Max surge pods during update |
| hpa.enabled | bool | false | Enable Horizontal Pod Autoscaler |
| hpa.minReplicas | int | 1 | Minimum replicas for HPA |
| hpa.maxReplicas | int | 10 | Maximum replicas for HPA |
| hpa.targetCPUUtilizationPercentage | int | 80* | Target CPU utilization (*default behavior) |
| hpa.targetMemoryUtilizationPercentage | int | null | Target memory utilization |
| pdb.enabled | bool | false | Enable PodDisruptionBudget |
| pdb.unhealthyPodEvictionPolicy | string | IfHealthyBudget | PDB eviction behavior |
| pdb.minAvailable | string/int | null | Minimum available pods |
| pdb.maxUnavailable | string/int | null | Maximum unavailable pods |
| service.enabled | bool | true | Create Service resource |
| service.type | string | ClusterIP | Service type |
| service.ports | list | [] | Service and container ports mapping |
| ingress.enabled | bool | false | Enable Ingress |
| ingress.ingressClassName | string | "" | IngressClass name |
| ingress.annotations | map | {} | Ingress annotations |
| ingress.hosts | list | [] | Ingress host rules |
| ingress.tls | list | [] | Ingress TLS configuration |
| httpRoute.enabled	| bool	| false	| Enable or disable HTTPRoute creation (Gateway API) |
| httpRoute.parentRefs	| list	| []	| Gateways to attach this route to, must provide at least one |
| httpRoute.hostnames	| list	| []	| Optional list of hostnames for routing |
| httpRoute.rules	| list	| []	| Ordered HTTP routing rules; each rule contains matches and backendRefs |
| httpRoute.rules[*].matches	| list	| []	| Conditions for matching requests (path, headers, etc.) |
| httpRoute.rules[*].backendRefs	| list	| []	| List of backends receiving traffic; supports name, port, weight, namespace |
| httpRoute.rules[*].backendRefs.name	| string	| Chart Service by default	| Backend Service name (defaults to chart’s Service if empty) |
| httpRoute.rules[*].backendRefs.port	| int/string	| First port in service.ports	| Backend Service port |
| serviceAccount.create | bool | false | Create ServiceAccount |
| serviceAccount.automount | bool | false | Automount ServiceAccount token |
| serviceAccount.annotations | map | {} | ServiceAccount annotations |
| serviceAccount.name | string | "" | ServiceAccount name |
| podAnnotations | map | {} | Annotations applied to pods |
| podLabels | map | {} | Labels applied to pods |
| podSecurityContext | object | {} | Pod-level security context |
| securityContext | object | {} | Container-level security context |
| volumes | list | [] | Additional volumes |
| volumeMounts | list | [] | Volume mounts for containers |
| nodeSelector | map | {} | Node selector labels |
| tolerations | list | [] | Pod tolerations |
| affinity | object | {} | Pod affinity / anti-affinity rules |
| priorityClassName | string | "" | PriorityClass name |
| restartPolicy | string | Always | Pod restart policy (Deployments require Always) |
| terminationGracePeriodSeconds | int | null | Pod termination grace period |
| initContainers | list | [] | Init containers |
| sidecars | list | [] | Sidecar containers |
