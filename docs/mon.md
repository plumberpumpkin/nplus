# nplus-component-mon

Extended nscale Monitoring Console Chart

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm install my-release nplus/nplus-component-mon
```

> Please read the documentation (like the **quick start guide**) before installing this chart. There are prerequisites which
> will make this release fail if not met.

## nplus-component-mon Chart Configuration

| Key | Description | Default |
|-----|-------------|---------|
image.name |  | `"monitoring-console"` |
image.repo |  | `"ceyoniq.azurecr.io/release/nscale"` |
image.tag |  | `"ubi.9.1.1000.2023091818"` |
ingress.backendProtocol |  | `"http"` |
ingress.enabled |  | `true` |
mounts.conf.path |  | `"/opt/ceyoniq/nscale-monitoring/workspace"` |
mounts.data.paths[0] |  | `"/opt/ceyoniq/nscale-monitoring/workspace/.metadata"` |
mounts.data.size |  | `"10Gi"` |
mounts.license.path |  | `"/opt/ceyoniq/nscale-monitoring/workspace/license.xml"` |
replicaCount |  | `1` |

## Common Image Configuration

The `image` configuration consists of

- the Image Name
- the Image Repository
- the Image Tag
- the Image Pull Policy

If the Pull Policy is not set, it is automatically `IfNotPresent`.

The `Repository` can be overridden at Instance Level and Environment Level to accomodate multiple stages:

```
image:
  name: test
  tag: 1.0.0
  repo: cr.nplus.cloud          # Prio 3
  pullPolicy: Always
global:
  repo: myrepo_i1               # Prio 4
  repoOverride: myrepo_i2       # Prio 2
  environment:
    repo: myrepo_e1             # Prio 5
    repoOverride: myrepo_e2     # Prio 1
```

In this example, finding the repo to use would be:

```helm
$repo := global.environment.repoOverride | default global.repoOverride | default image.repo | default global.repo | default global.environment.repo
```

**The Use Case** is to easily enable you to download the images to a private and secure registry. *nplus* by default uses the official registries, but
that is most likely not wanted by enterprise customers. So you can just set your own registry in the environment and keep dev, qa and prod apart and secured.

## Common Ingress Configuration

The Ingress Configuration can be performed at various levels:

- Per Component / Chart
  `ingress.`
- Per Instance
  `global.ingress.`
- Per Environment
  `global.environment.ingress.`

This enables you to have configuration yaml files per environment (e.g. for DEV, QA and PROD) setting environment defaults.
You then do not have to touch the Instance configuration.

Example:

```
helm upgrade -i \
    --values $SAMPLES/big-instance.yaml \
    --values $SAMPLES/applications.yaml \
    --values $SAMPLES/dev.yaml \
    demo nplus/instance-argo
```

You might have your Instance values in the `big-instance` file, the Apps you want to have deployed to that instance
in the `applications` file, and then you add your default setting for the `dev` stage, potentionally overwriting anything
from the above. The priority in this is *last one wins*.

> The Values are taken by the chart in the following order:
> Component, then Instance, then Environment.

If no value is set, the configuration is dropped from the manifest.

In the following table, you see what value can be defined in which section:

| Key  | Component  | Instance  | Environment |
| ---- | ----------- | ----------- | ----------- |
|domain | ✔︎ | ✔︎ |✔︎|
|issuer | ✔︎ | ✔︎ |✔︎|
|proxyip | ✔︎ | ✔︎ | `0.0.0.0/0` |
|class | ✔︎ | ✔︎ | `public` |
|enabled | ✔︎ | - |-|
|backendProtocol | ✔︎ | - |-|
|cookie | ✔︎ | - |-|
|inputPath | ✔︎ | - |-|

For the component ingress, you can specify the following values:

| Key | Description | Default |
|-----|-------------|---------|
backendProtocol | choose wether you want http or https as the backend protocol. This will encrypt traffic from the ingress controller to your pods if you set it to https. | `"http"` |
class | sets the ingressclass to use. e.g. `public` or `nginx` | `"public"` |
cookie | on component level, set cookie affinity for the ingress example: `XtConLoadBalancerSession` for nscale Web | component dependent |
domain | sets the ingress domain, like `tenant1.mydomain.com`. If no domain is set, no ingress will be configured automatically | none |
enabled | on component level, enable or disable the ingress | component dependent |
inputPath | this defines the path (on component level) for this component Example: `nscale_web` for nscale Web | component dependent |
issuer | if you use *cert-manager* or any other certificate issuer, you can add the class here to hand certificate issuing requests to this issuer. if you do not set any issuer, the chart will generate a self-signed certificate for your ingress (if you defined a domain) |  |
proxyip | if you use a reverse proxy in front of your cluster, allow incoming traffic from this IP range | `"0.0.0.0/0"` |

## Common Storage Configuration

This works just the same way as the Ingress settings: The Configuration can be performed at various levels:

- Per Component / Chart
  `storage.`
- Per Instance
  `global.storage.`
- Per Environment
  `global.environment.storage.`

This enables you to have configuration yaml files per environment (e.g. for DEV, QA and PROD) setting environment defaults.
You then do not have to touch the Instance configuration.

For storage, there are several volume types:

- **conf**, Shared File, RWX, global per environment
- **data**, Disk, RWO, optional per component
- **file**, Shared File, RWX, optional per ReplicaSet
- **temp**, EmptyDir
- **log**, EmptyDir, should be empty, so just in case
- **pool**, optional path on the conf share mounted by some components

### conf

The *conf* storage is a global PVC with RWX (file) shared by every component in the environment. The component creates a sub directory
on the share and mounts it to the config directory in the container.

`storage.conf.name` sets the name of the PVC to be created and used.
`mounts.conf.path` defines the target directory in the container.

As the environment normally provides the *conf* share, you can set the class and the size in the environment.

### data

Every component can create a data PVC with RWO (disk). You can set the `class` for this disk directly at the mount definition `mounts.data.class`. If unset, it uses the definition for the data class from `global.storage.data.class` or from the environment definition at `global.environment.storage.data.class`.

If the class is not defined, it is not included in the manifest and so the cluster default is taken.

Set the size at `mounts.data.size`.  No default for the size.

### file

Every component can create a file PVC with RWX (shared file). You can set the `class` for this share directly at the mount definition `mounts.file.class`. If unset, it uses the definition for the file class from `global.storage.file.class` or from the environment definition at `global.environment.storage.file.class`.

If the class is not defined, it is not included in the manifest and so the cluster default is taken.

Set the size at `mounts.file.size`.  No default for the size.

This file mount is used for example for the *nscale Rendition Server* to create a common workload directory for all PODs across cluster nodes.

### temp

If a *temp* mount point is given in the values file, it creates an `emptyDir` volume with the `sizeLimit` of `mounts.temp.size`. If no limit is given, the volume will have no limit and the cluster node default is used.

If you want to back this volume by memory, specify `mounts.temp.medium: memory`. Be aware, that this will utilize a RAM disk and count against your PODs ressources.

> The *nscale Application Layer* caches fulltext data in temp. Please be aware of your component behaviour when setting medium and size. Your plugins might be requireing speed or size.

### logs

If a *logs* mount point is given in the values file, it creates an `emptyDir` volume with the `sizeLimit` of `mounts.logs.size`. If no limit is given, the volume will have no limit and the cluster node default is used.

The components are writing logs to `stdout` and `stderr`, so the logs directory should not be necessary. This is just in case any plugin writes something to the contaainers file system.

Additionally, if you use the *nplus Administrator Server* component, you might want the legacy way of reading log files, and this would be the storage for that.

### pool

You can define a path at `mount.pool`, then this component will have access. This is used to hand binary data to the components, such as plugins or *nscale Generic Base Apps* along with the *nscale App Installer*.

### Setting storage values

| Key  | Component  | Instance  | Environment |
| ---- | ----------- | ----------- | ----------- |
| conf.name | ✔︎ | ✔︎ | `conf` |
| data.class | ✔︎ | ✔︎ | ✔︎ |
| data.size | ✔︎ | - | - |
| data.paths | predefined list | - | - |
| file.class | ✔︎ | ✔︎ | ✔︎ |
| file.size | ✔︎ | - | - |
| file.paths | predefined list | - | - |
| temp.size | ✔︎ | - | - |
| temp.medium | ✔︎ | - | - |
| temp.path | predefined | - | - |
| logs.size | ✔︎ | - | - |
| logs.medium | ✔︎ | - | - |
| logs.path | predefined | - | - |

Avoid to change the values marked as *predefined*.

## Security settings

You can set the security options per *component*, per *instance* or per *environment*.
The priority is:

1. component
2. instance
3. environment

It is recommended to set the security per environment to make sure you do not forget a component.

### Illumio

Example `environment` setting for Illumio:

```
global:
  environment:
    security:
      illumio:
        enabled: true
        loc: "mylocation"
        supplier: "mysupplier"
        platform: "myplatform"
        readinessGates:
          - conditionType: "com.illumio.policy-ready"
```

### Calico

Example `environment` setting for Calico:

```
# TODO
```

### Security Conext

You can add a `containerSecurityContext` to the component by adding it in the values file:

```
security:
  containerSecurityContext:
    capabilities:
      drop: ["ALL"]
```

Additionally, add a `podSecurityContext` if desired:

```
security:
  podSecurityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
```

> **Note**: This setting can not be set on instance or environment level.

## Ressources

### Kubernetes

By default, no resources are set on the container. Thus, Kubernetes handles the container with best effort.

Ressources can be set at

| Key | Description | Default |
|-----|-------------|---------|
| ressources.requests.cpu | sets the request, which is the minimum guaranteed | - |
| ressources.requests.memory | sets the request, which is the minimum guaranteed | - |
| ressources.limits.cpu | sets the limit, which is the maximum allowed | - |
| ressources.limits.memory | sets the request, which is the maximum allowed | - |

- if nothing is defined, Kubernetes handles it BestEffort
- if requests are defined, but no limits, Kubernetes handles it Burstable
- if both are defined, Kubernetes handles it Guaranteed

Please take caution when setting parameters and also have a look at this interesting article regarding resources and JVM resource handling:

https://xebia.com/blog/kubernetes-and-the-jvm/

### JVM

For those components implemented in Java, it is possible to set Java Options:

- nscale CMIS Connector
- nscale ILM Connector
- nscale Application Layer
- nscale Rendition Server
- nscale Web

| Key | Description | Default |
|-----|-------------|---------|
| javaOpts.javaMaxRamPercentage | Maximum memory given to Java in % | - |
| javaOpts.javaMinMem | Minimum memory given to Java | - |
| javaOpts.javaMaxMem | Maximum memory given to Java | - |
| javaOpts.javaMisc | Additional Java Options | - |

> **Note**: if you defined settings for *appDynamics*, the agent will automatically be added to the Java Options when the above components are run. Please refer to `global.appDynamics.agent` for more information.
