# moc-services-config

This repository manages the configuration of the `moc-services` [EKS] cluster. We use [ArgoCD] to automatically apply repository changes to the cluster.

## Cluster services

The moc-services cluster provides:

- [Keycloak], our IdP for the Open Accelerator OpenShift clusters

Support services include:

- [Postgres], managed by the [Crunchy Data Postgres Operator][pgo]
- [Cert-manager], for generating self-signed certificates
- [External secrets], for pulling secrets from AWS Secrets Manager and for copying secrets between namespaces

[EKS]: https://aws.amazon.com/eks/
[pgo]: https://github.com/crunchydata/postgres-operator
[cert-manger]: https://cert-manager.io/
[external secrets]: https://external-secrets.io/

## Working with ArgoCD

We deploy a single [ApplicationSet] from [`root.yaml`](base/applicationsets/root.yaml), patched for a specific deployment target (AWS or [Kind]). The ApplicationSet creates an ArgoCD [Application] for every directory in `overlays/<TARGET>`. For AWS, this means (as of this writing):

[applicationset]: https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/
[application]: https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#applications

- `applicationsets`
- `argocd`
- `cert-manager`
- `external-secrets`
- `keycloak`
- `metrics-server`
- `pgo`
- `postgres-common`

You can monitor the status of the applications using the [`argocd` cli][cli]. To see the status of all applications:

[cli]: https://argo-cd.readthedocs.io/en/stable/cli_installation/

```sh
$ argocd app list
NAME                     CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                PATH                           TARGET
argocd/applicationsets   https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/applicationsets   main
argocd/argocd            https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/argocd            main
argocd/cert-manager      https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/cert-manager      main
argocd/external-secrets  https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/external-secrets  main
argocd/keycloak          https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/keycloak          main
argocd/metrics-server    https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/metrics-server    main
argocd/pgo               https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/pgo               main
argocd/postgres-common   https://kubernetes.default.svc             default  Synced  Healthy  Auto        <none>      https://github.com/cci-moc/moc-services-config.git  overlays/aws/postgres-common   main
```

To see the resources involved in a single application:

```sh
$ argocd app get postgres-common
Name:               argocd/postgres-common
Project:            default
Server:             https://kubernetes.default.svc
Namespace:
URL:                http://localhost:40781/applications/postgres-common
Source:
- Repo:             https://github.com/cci-moc/moc-services-config.git
  Target:           main
  Path:             overlays/aws/postgres-common
SyncWindow:         Sync Allowed
Sync Policy:        Automated
Sync Status:        Synced to main (bc6d248)
Health Status:      Healthy

GROUP                              KIND                NAMESPACE  NAME                              STATUS  HEALTH   HOOK  MESSAGE
                                   Namespace                      postgres                          Synced                 namespace/postgres serverside-applied
rbac.authorization.k8s.io          ClusterRole                    external-secrets-postgres-reader  Synced                 clusterrole.rbac.authorization.k8s.io/external-secrets-postgres-reader serverside-applied
rbac.authorization.k8s.io          RoleBinding         postgres   external-secrets-postgres-reader  Synced                 rolebinding.rbac.authorization.k8s.io/external-secrets-postgres-reader serverside-applied
external-secrets.io                ClusterSecretStore             postgres                          Synced  Healthy        clustersecretstore.external-secrets.io/postgres serverside-applied
postgres-operator.crunchydata.com  PostgresCluster     postgres   postgres                          Synced  Healthy        postgrescluster.postgres-operator.crunchydata.com/postgres serverside-applied
```

ArgoCD also provides a web interface. The easiest way to access it is to set up port forwarding from your local machine to the `argocd-server` service:

```sh
oc port-forward -n argocd svc/argocd-server 8080:80
```

While that command is running, the ArgoCD UI will be available at <http://localhost:8080>. You will log in as user `admin`. You can obtain the admin password by running:

```sh
oc -n argocd extract secret/argocd-initial-admin-secret --to=-
```

This will print the password on your console.

## License

[Apache 2.0 License](LICENSE).

The code is provided as-is with no warranties.
