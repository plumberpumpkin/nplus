# Install, Update, Uninstall

1. Install instance *sample*

    To demonstrate, we use the sample-tenant chart we find in the samples directory. The main difference
    to the default instance chart is, that a domain is set to `*.sample.nplus.cloud`, so we will be able to
    log into the web client right away if we redirected this domain correctly.

    You can easily adopt the examples to your environment.

    ```
    helm install sample nplus/sample-tenant --version 9.0.1400
    ```

2. **Rolling update** of instance *sample* to a later monthly release

    All nscale components support rolling updates, **but** the *nscale Application Layer*.
    As the Application Layer has the connection to the database, and this depends on the DB scheme,
    only cluster members with the same version can work with that DB at the same time.

    There are no scheme updates in monthly releases, so we can use the default rolling updates here.

    ```
    helm upgrade sample nplus/sample-tenant --version 9.0.1501
    ```

3. **Minor / Major Update** of instance *sample*

    Minor or Major updates require the *nscale Application Layer* to have the same version on all cluster nodes. And since the *nscale Pipeliner* may also have an integrated *nappl* in core mode, we also need to update the pipeliner at the same time.

    We first need to shut down all *nappl* cluster members, so set the *nscale Application Layer*, the potential *nappl Jobs Node* and the *nscale Pipeliner* stateful sets to replica 0.

    In *nplus*, these replicaSets are labeled with `nplus/type=nappl`, so we can easily select them:

    ```
    kubectl scale statefulset -l nplus/type=nappl,nplus/instance=sample --replicas=0
    ```

    After that, the update is just like a monthly release:

    ```
    helm upgrade sample nplus/sample-tenant --version 9.1.1001
    ```

5. **Uninstall** the instance *sample*
   
    ```
    helm uninstall sample
    ```



# Install, Update, Uninstall *with argoCD*

1. Install instance *sample-argo*

    ```
    helm install sample-argo nplus/sample-tenant-argo --version 9.0.1400
    ```


2. **Rolling update** of instance *sample-argo* to a later monthly release

    ```
    helm upgrade sample-argo nplus/sample-tenant-argo --version 9.0.1501
    ```


3. **Minor / Major Update** of instance *sample-argo*

    The difference to a deployment without argoCD is, that if we manually scale down the *nappl* cluster nodes,
    argoCD tries to immediately **heal** this discrepancy between the description and the status.
    So we first switch off this healing mechanism, to be able to scale down:

    ```
    kubectl -n argocd  patch --type='merge' application sample-argo -p "{\"spec\":{\"syncPolicy\":null}}" 
    ```

    After that, it is the same update procedure as we have with a standard deployment:

    ```
    kubectl scale statefulset -l nplus/type=nappl,nplus/instance=sample-argo --replicas=0
    helm upgrade sample-argo nplus/sample-tenant-argo --version 9.1.1001
    ```

    When done, we switch the healing back on which will start to re-sync and recreate all cluster members
    with the new version:

    ```
    kubectl -n argocd patch --type=merge application sample-argo -p "{\"spec\":{\"syncPolicy\":{\"automated\":{\"prune\":true,\"selfHeal\":true}}}}"
    ```


5. **Uninstall** the instance *sample-argo*
   
    ```
    helm uninstall sample-argo
    ```


