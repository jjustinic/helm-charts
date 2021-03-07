# Workload Configuration

Kubernetes Workload resources:

* [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

are created depending on configuration values.

## Global Section

Default yaml defined in the product workload section.

```yaml
global:
  workload:
    type: Deployment

    deployment:
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 0

    statefulSet:
      partition: 0

      persistentvolume:
        enabled: true
        volumes:
          out-dir:
            mountPath: /opt/out
            persistentVolumeClaim:
              accessModes:
              - ReadWriteOnce
              storageClassName:
              resources:
                requests:
                  storage: 4Gi

    securityContext: {}

    serviceAccount:
      generate: false
```

| Workload Parameters               | Description                                                                                  |
| --------------------------------- | -------------------------------------------------------------------------------------------- |
| type                              | One of Deployment or StatefulSet                                                             |
| deployment.strategy.type          | One of RollingUpdate or ReCreate                                                             |
| deployment.strategy.rollingUpdate | If type=RollingUpdate                                                                        |
| statefulSet.partition             | Used for canary testing if n>0                                                               |
| statefulSet.persistentVolume      | Provides details around creation of PVC/Volumes (see below)                                  |
| securityContext                   | Provides security context details for starting container as different user/group (see below) |
| serviceAccount.generate           | Enables service account definition and usage                                                          |

!!! note "Persistent Volumes"
    For every volume defined in the volumes list, 3 items will be
    created in the StatefulSet:

    * container.volumeMounts - name and mountPath
    * template.spec.volume - name and persistentVolumeClaim.claimName
    * spec.volumeClaimTemplates - persistentVolumeClaim

    More Info - <https://kubernetes.io/docs/concepts/storage/persistent-volumes/>

!!! note "Security Context"
    To run the containers with a different user/group/fsgroup, use the following
    example to set those details on the deployment/statefulset:

    ```yaml
    global:
      workload:
        container:
          securityContext:
            runAsGroup: 1000
            runAsUser: 3000
            fsGroup: 2000
    ```

!!! note "Service Account"
    To generate and run with a unique service account per workload, enable this
    option. The generated service account will be used over any other
    `serviceAccountName` configuration.

!!! note "WaitFor"
    For each product, a `waitFor` structure providing the name, service and timeout, in seconds,
    that should be waited (defaults to 300 if not provided) on before the running container con continue.  This
    will inject an initContainer using the PingToolkit wait-for utility until it
    can `nc host:port` before continuing.

    Example: PingFederate Admin waiting on pingdirectory ldaps service to be available
    ```yaml
    pingfederate-admin:
      container:
        waitFor:
          pingdirectory:
            service: ldaps
            timeoutSeconds: 600
          pingdatagovernance:
            service: https
            timeoutSeconds: 300
    ```

    * By default, the `pingfederate-engine` will waitFor `pingfederate-admin` before it starts.
    * By default, the `pingaccess-engine` will waitFor `pingaccess-admin` before it starts.