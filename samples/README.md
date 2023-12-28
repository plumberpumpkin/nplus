# Configuration Options

There are several possible ways to configure and customize the *nplus* helm charts:

1. By using `--set` parameter at the command line:

  ```
  helm install \
    --set components.rs=false \
    --set components.mon=false \
    --set components.nstl=false \
    --set global.ingress.domain="demo.nplus.cloud" \
    --set global.ingress.issuer="nplus-issuer" \
    demo1 nplus/nplus-instance
  ```

2. By adding one or more values files:

  ```
  helm install \
      --values empty.yaml \
      --values s3.yaml \
      --values lab.yaml \
      demo2 nplus/instance
  ```

3. By using `helm template` and piping the output to `kubectl`

  ```
  helm template \
      --values empty.yaml \
      --values s3.yaml \
      --values lab.yaml \
      demo2 nplus/instance      | kubectl apply -f -
  ```

4. By building *Umbrella Charts* that contain default values for your environment

See these sample charts:

- [servicegroup](servicegroup)
  is a sample chart that customizes the default 
  *instance* chart with extra default parameters
- [servicegroup-argo](servicegroup-argo)
  is a sample chart that customizes the default 
  *instance-argo* chart with extra default parameters

# Sample Value Files

The sample value files provided here work for standard Instances as well as for argoCD Versions.

The demo site at [https://demo.nplus.cloud](https://empty.demo.nplus.cloud) has been built with these scripts.

The values Files are stacked:
- `demo.yaml` contains the default values for the environment
- `empty.yaml` creates a sample document area in the *nscale Application Layer*
- `sbs.yaml` installs the *Smart Business Solutions*
- `s3.yaml` adds a S3 storage to the *nscale Storage Layer*

These are the samples:

- Environment Sample
  You need one *nsplus Environment* per K8s namespace. You can install it using the same yaml file you would use for the instances:
  ```
  helm install \
    --values samples/demo.yaml \
    demo nplus/nplus-environment
  ```

- Empty Instance Sample
  This is the simplest sample, just the core services with an empty document area:
  ```
  helm install \
    --values samples/empty.yaml \
    --values samples/demo.yaml \
    empty nplus/nplus-instance
  ```

- SBS Sample
  Installs an Instance and deploys SBS to a document area
  ```
  helm install \
    --values samples/sbs.yaml \
    --values samples/demo.yaml \
    sbs nplus/nplus-instance
  ```

- All In One Sample
  Shows a multi replica HA Instance
  ```
  helm install \
    --values samples/demo-all-in-one.yaml \
    --values samples/empty.yaml \
    --values samples/demo.yaml \
    demo-all-in-one nplus/nplus-instance
  ```

- Central Services Sample
  Some organisations have multiple tenants that share common services, like *nscale Rendition Server* or
  have a common IT department, thus using only a single *nscale Monitoring Console* acress all tenants.
  This is the Central Services Part:
  ```
  helm install \
    --values samples/demo-centralservices.yaml \
    --values samples/demo.yaml \
    demo-centralservices nplus/nplus-instance
  ```
  And this is the tenant using the Central Services:
  ```
  helm install \
    --values samples/demo-shared.yaml \
    --values samples/empty.yaml \
    --values samples/demo.yaml \
    demo-shared nplus/nplus-instance
  ```

When deploying using argoCD, you can use the same files but just add an `*-argo` to the chart name:

- Empty Instance Sample with argoCD
  ```
  helm install \
    --values samples/empty.yaml \
    --values samples/demo.yaml \
    empty-argo nplus/nplus-instance-argo

And so forth...



# Check whats deployed

```
% kubectl get nplus
NAME                        HANDLER   VERSION    CHART   GEN   COMPONENTS
default                     Helm      9.1.1201   1.0.3   3     database,nappl,nstl,rs,web
demo-all-in-one             Helm      9.1.1201   1.0.3   3     application,cmis(2),database,ilm(2),mon,nappl(2),nappljobs,nstl,rs(2),web(2)
demo-centralservices        Helm      9.1.1201   1.0.3   3     mon,nstl,rs
demo-shared                 Helm      9.1.1201   1.0.3   3     application,database,nappl,web
empty                       Helm      9.1.1201   1.0.3   3     application,database,nappl,nstl,rs,web
tenant                      Helm      9.1.1201   1.0.3   3     application,database,nappl,nstl,rs,web
sbs                         Helm      9.1.1201   1.0.3   4     application,database,nappl,nstl,rs,web
demo-centralservices-argo   argoCD    9.1.1201   1.0.3   3     mon,nstl,rs
demo-shared-argo            argoCD    9.1.1201   1.0.3   3     application,database,nappl,web
empty-argo                  argoCD    9.1.1201   1.0.3   3     application,database,nappl,nstl,rs,web
demo-all-in-one-argo        argoCD    9.1.1201   1.0.3   3     application,cmis(2),database,ilm(2),mon,nappl(2),nappljobs,nstl,rs(2),web(2)
tenant-argo                 Helm      9.1.1201   1.0.3   3     application,database,nappl,nstl,rs,web
sbs-argo                    argoCD    9.1.1201   1.0.3   3     application,database,nappl,nstl,rs,web
```