# *nplus* Quickstart Guide

The charts are built in a way that they will provide minimal functionality without any configuration using default values.

- if you want ingress, you have to configure the domain  
  without the domain set, your charts will not have any default way to access them.
  However, you can still forward traffic to them our configure a *NodePort* or *LoadBalancer* manually.
- if your want proper tls, you need a certificate.
  withour the certificate provided, a self-signed certificate will secure your connection.
- if you want specific storage, configure the storage class to use.
  without, you will get the default class for RWO and RWX

This Quick Start example has nothing configured, so you will get

- no ingress, and
- default storage



## Infrastructure

It is assumed in this tutorial, that you have a running hardware with network and a linux system installed with snap capability.

In terms of hardware, I am testing this minimal setup with a PC with 4 cores, 8 GB RAM, 32 GB disk.



## Set up a Kubernetes Cluster

For a demo environment, we recommend *microk8s*. Our examples will be using *microk8s*, but they should be running with any comatible Kubernetes deployment.

We will need *curl* and *sudo* (needed by microk8s installer).
Microk8s comes as a snap, so we also need to install *snapd*.

```
apt-get update && apt-get -y install curl sudo snapd
```

Then we can install *microk8s*:

```
snap install microk8s --classic --channel latest/stable
```

After installation, *microk8s* can be started with

```
microk8s start
```

Check the status with

```
# microk8s status
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
  disabled:
    cert-manager         # (core) Cloud native certificate management
    community            # (core) The community addons repository
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    ingress              # (core) Ingress controller for external access
    metallb              # (core) Loadbalancer for your Kubernetes cluster
    rook-ceph            # (core) Distributed Ceph storage using Rook
...
```

For a first install, we need some storage. We recommend *ceph* for a productive environment, but for a small demo server with a single node, *hostpath* is ok for now.

More is not required in this example, but it is a good idea to also enable *ingress*.

```
microk8s enable hostpath-storage
microk8s enable ingress
```



## Set up the client

Depending on your operating system, please install *kubectl* and *helm3*:
- [kubectl](https://kubernetes.io/de/docs/tasks/tools/)
- [helm3](https://helm.sh/docs/intro/install/)

The client needs the servers configuration in order to access. *Microk8s* can generate this configuration for you: (run from client)

```
mkdir -p ~/.kube
ssh [IP of your Sever] /snap/bin/microk8s kubectl config view --raw > ~/.kube/config.demo
```

> Please check if the IP Adress is correct in the file `~/.kube/config.demo`. It should be the Adress of your new Kubernetes server, not 127.0.0.1. You might need to correct that.

Now we need to tell *kubectl* that there is a new cluster configuration in `~/.kube/config.demo`. This can be done easily by setting the `KUBECONFIG` environment variable. You can have multiple config files.

I like to add a little function to the `.profile` to grab all new configs when I log in:

```
export KUBECONFIG=$(for YAML in $(find ${HOME}/.kube/ -name 'config.*') ; do echo -n ":${YAML}"; done) 
```

So once set, you can tell *kubectl* to use the new cluster:

```
% kubectl config get-contexts -o name
microk8s
```

```
kubectl config use-context microk8s
```

Now we can check if we can access the cluster from our client:

```
% kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
k-demo-n1   Ready    <none>   20m   v1.28.3
```



## Access to the *nplus* subscription and the nscale license

You need access to

- the *nplus* helm chart repository
- the *nplus* container registry

- the *nscale* license
- the *nscale* container registry

In the next examples, we will use enviroment variables to access. For now, it is OK to just get them into your current session. 

> We do recommend though, to create a directory `~/.nplus/[context]` and place them there, as scripts will later on look for e.g. `~/.nplus/microk8s/config` (depending on your current context). This way, you can switch easily beween different Kubernetes Clusters and have different license information for each one.

```
NPLUS_ACCOUNT="[your nplus subscription]"
NPLUS_TOKEN="[your nplus access token]"
NSCALE_ACCOUNT="[your account to access the Ceyoniq container registry]"
NSCALE_TOKEN="[the access token for above]"
NSCALE_LICENSE="[the path and license file to use]"
```

Now you can register the *nplus* helm registry

```
helm repo add nplus https://helm.nplus.cloud \
	--username $NPLUS_ACCOUNT \
	--password $NPLUS_TOKEN
helm repo update
```

You should now be able to access the charts:

```
% helm search repo nplus --versions --devel
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                                       
gitea/nplus-application        	9.1.1201-16  	0.2.2      	Application Chart                                 
gitea/nplus-application        	9.1.1201-15  	0.2.2      	Application Chart                                 
gitea/nplus-application        	9.1.1201-14  	0.2.2      	Application Chart                                 
...
```

> the `--devel` option gives you development versions as well. Otherwise you will only see release version.



## Create a nplus Environment

You can deploy nplus into a Kubernetes namespace. If you do not specify one, you will use the default one, which is fine for our test cluster. If you use namespaces, you can have multiple *nplus* environments in your cluster. Any environment can operate multipe *nplus* instances. Evenry *nplus* instance normaly holds many components, each being *ReplicaSets* with multiple replicas.

To create a simple *nplus* environment without any additional features, just deploy it into your new cluster:

> by setting `--devel`, we are fetching the latest development version


```
% helm install --devel demo nplus/nplus-environment
NAME: demo
LAST DEPLOYED: Tue Dec 19 16:39:51 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
nplus-environment 0.2.2-16 / 0.2.2

This Environment Chart provides a common config pool and administrative tools to operate all nplus instances in this namespace. There must be exactly one deployed  instance of this environment chart. Without the environment, the instance and component charts will fail to deploy. 

To uninstall, use
   helm uninstall demo

The environment DAV Server is disabled.
The nstore Downloader is disabled.
The toolbox is disabled.

Providing 10Gi of storage under the name "conf" of class "default"
```

Now you have an empty cluster ready to get a first instance deployment



## Deploy a nplus Instance

Before we can deploy the first *nplus* Instance, we need to add the Secrets for the registries and also the nscale license to the environment:

```
kubectl create secret docker-registry nscale-cr \
  --docker-server=ceyoniq.azurecr.io \
  --docker-username=$NSCALE_ACCOUNT \
  --docker-password=$NSCALE_TOKEN

kubectl create secret docker-registry nplus-cr \
  --docker-server=cr.nplus.cloud \
  --docker-username=$NPLUS_ACCOUNT \
  --docker-password=$NPLUS_TOKEN

kubectl create secret generic nscale-license \
  --from-file=$NSCALE_LICENSE 
```

Secrets are namespace dependent (one cannot access secrets from other namespaces), so we have to deploy them for every environment / namespace we use in our cluster.

There are multiple ways of deploying a *nplus* Instance, the easiest one is by simply calling the helm install on the command line:

```
helm install --devel myinstance nplus/nplus-instance
```

After a couple of minutes, you can check if the instance came up

```
% kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
myinstance-database-0             1/1     Running   0          10m
myinstance-nstl-0                 1/1     Running   0          10m
myinstance-rs-8b859fbbc-j5h4r     1/1     Running   0          10m
myinstance-nappl-0                1/1     Running   0          10m
myinstance-web-7579c6d7dd-rfckh   1/1     Running   0          10m
```

You can check the log files of the *Application Layer* for instance by typing

```
kubectl logs -l nplus/instance=myinstance,nplus/component=nappl
```

> Notice the locater in the logs example: Instead of telling kubectl the name of the pod or rs, we use locaters, because there may be multiple instances of these pods later and we want to see all logs in one go (or have ELK, EFK, slpunk or anything similar to do that for us)






---
**Now this is it: We have a running *nplus* Instance.**

---






## A first configuration: Opening an ingress

We need to know the available ingressClasses in our new Kubernetes Cluster, so we check that:

```
% kubectl get ingressclass
NAME     CONTROLLER             PARAMETERS   AGE
public   k8s.io/ingress-nginx   <none>       72m
nginx    k8s.io/ingress-nginx   <none>       72m
```

*Microk8s* comes with the most common classes, which both point to the same controller (in this case nginx). *public* is in deed the default class for *nplus*. So we do not need to set that, it is already configured. We just need to tell the *nplus* instance to use a Domain for the ingress:

```
helm upgrade --devel \
  --set global.ingress.domain=myinstance.demo.nplus.cloud \
  myinstance nplus/nplus-instance
```

This now activates an ingress for [https://myinstance.demo.nplus.cloud/nscale_web](https://myinstance.mydomain.demo.nplus.cloud/nscale_web). The easiest and fastest is probably to add the IP to the server into your `/etc/hosts` file. 

> The browser will complain about the self signed certificate. You can easily add your own certificate into the secret `myinstance.demo.nplus.cloud-tls` which has been created for you.
> However, the canonical way is to have *cert-manager* or a similar tool take care of your certificates and have them generated by your own CA or *Lets Encrypt* or similar.



## Further Reading

please have a look at the README.md of the Chart to explorer more configuration options:

```
helm show readme nplus/nplus-environment
helm show readme nplus/nplus-instance
```

There are also charts for every component used by the instance umbrella chart.



## More Configuration Options

You can also start configuring you instance by retrieving and altering the values.yaml of the chart.

Example:

```
helm show values  --devel nplus/nplus-instance > myinstance.yaml
```

Then edit this file.

When you are done, apply it:

```
helm upgrade --devel \
  -f myinstance.yaml \
  myinstance nplus/nplus-instance
```

> Please be aware, that the umbrella `values.yaml` does **not** contain all possible configuration options of the child charts.

