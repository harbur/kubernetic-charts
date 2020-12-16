# Kubernetic Chart

Kubernetic provides is a single pane of glass for managing Kubernetes clusters, for both operators and developers.

This chart installs the Kubernetic components to the cluster, a backend and a frontend, to provide access through the web.

## Get Repo Info

```sh
helm repo add kubernetic https://charts.kubernetic.com
helm repo update
```

See [helm repo] for command documentation.

[helm repo]: https://helm.sh/docs/helm/helm_repo/

## Install Chart

```sh
# Helm 3
$ helm install [RELEASE_NAME] kubernetic/kubernetic [flags]
```

See [configuration] below.

See [helm install] for command documentation.

[configuration]: #configuration
[helm install]: https://helm.sh/docs/helm/helm_install/

## Uninstall Chart

```sh
# Helm 3
$ helm uninstall [RELEASE_NAME]
```

This removes all the Kubernetes components associated with the chart and deletes the release.

See [helm uninstall] for command documentation.

[helm uninstall]: https://helm.sh/docs/helm/helm_uninstall/

## Upgrade Chart

```sh
# Helm 3
$ helm upgrade [RELEASE_NAME] kubernetic/kubernetic [flags]
```

See [helm upgrade] for command documentation.

[helm upgrade]: https://helm.sh/docs/helm/helm_upgrade/

## Configuration

See [Customizing the Chart Before Installing]. To see all configurable options with detailed comments, visit the chart's [values.yaml](./values.yaml), or run these configuration commands:

```sh
# Helm 3
$ helm show values kubernetic/kubernetic
```

For a summary of all configurable options, see [Chart Values].

[Customizing the Chart Before Installing]: https://helm.sh/docs/intro/using_helm/#customizing-the-chart-before-installing
[Chart Values]: #Chart-Values

### Configure Image Repository

In case of private clusters without external access, you can configure a custom repository to use for pulling the images from there:

```yaml
backend:
  image:
    repository: PRIVATE_REGISTRY/kubernetic-backend
frontend:
  image:
    repository: PRIVATE_REGISTRY/kubernetic-frontend
```

### Configure PVC

A PVC is used by backend in order to keep preferences and licensing. You can disable it using the following configuration, but keep in mind that this will make preferences and licensing ephemeral and would need reconfiguration after a pod restart.

```yaml
persistence:
  enabled: false
```

In case you want to manually configure the PVC before installing the helm chart, you can pass the pre-existing PVC (e.g. `kubernetic-backend-pvc`) in the configuration:

```yaml
persistence:
  existingClaim: kubernetic-backend-pvc
```

### Configure RBAC

Kubernetic uses impersonation to provide access to users with their corresponding privileges. By default it is provided ClusterRoleBinding with `cluster-admin` ClusterRole to the ServiceAccount used by the backend.

If you want to restrict Kubernetic access to a specific Namespaces you can disable RBAC and configure a RoleBinding on each namespace:

```yaml
rbac:
  create: false
```

```sh
> kubectl create rolebinding admin --clusterrole=admin --namespace KUBERNETIC_NAMESPACE
```

> If Kubernetic doesn't have privileges to list namespaces it will show an error message: `namespaces is forbidden: User "system:serviceaccount:****:****" cannot list resource "namespaces" in API group "" at the cluster scope`. To continue, close the message and open the namespaces dropdown, fill-in manually the namespace with the granted privileges and you will be able to operate within the namespace.

### Configure External URL

DO NOT EXPOSE DIRECTLY TO PUBLIC URL: Kubernetic assumes that Single Sign On (SSO) is configured on top and handles authentication, if you expose the services directly to the public, you risk exposing your cluster.

Chart doesn't provide any mechanism exposing to the public, instead you should configure your SSO for authentication.

To test Kubernetic instance without exposing you can do port-forwarding to the service `kubernetic-frontend`:

```sh
$ kubectl port-forward svc/kubernetic-frontend 8888:80
Forwarding from 127.0.0.1:8888 -> 80
Forwarding from [::1]:8888 -> 80
```

then, open on a browser http://localhost:8888

We provide below an example how to setup SSO using Pomerium on top of Kubernetic.

### Pomerium SSO


## Chart Values

### Backend

| Parameter                   | Description                                        | Default                                                                        |
|-----------------------------|----------------------------------------------------|--------------------------------------------------------------------------------|
| backend.image.repository    | Backend image repository                           | `europe-west1-docker.pkg.dev/woven-computing-234012/public/kubernetic-backend` |
| backend.image.tag           | Backend image tag                                  | `3.1.0`                                                                        |
| backend.image.pullPolicy    | Backend image pull policy                          | `IfNotPresent`                                                                 |
| backend.updateStrategy      | TBD                                                |                                                                                |
| backend.imagePullSecrets    | Backend image pull secrets                         | `[]`                                                                           |
| backend.resources           | Backend resources allocation (Requests and Limits) | `{}`                                                                           |
| backend.nodeSelector        | Backend node labels for pod assignment             | `{}`                                                                           |
| backend.tolerations         | Backend toleration labels for pod assignment       | `[]`                                                                           |
| backend.affinity            | Backend affinity settings                          | `{}`                                                                           |
| backend.annotations         | Backend deployment annotations                     | Not Set                                                                        |
| backend.labels              | Backend deployment labels                          | Not Set                                                                        |
| backend.podAnnotations      | Backend pod annotations                            | Not Set                                                                        |
| backend.podLabels           | Backend pod labels                                 | Not Set                                                                        |
| backend.service.type        | Backend service type                               | `ClusterIP`                                                                    |
| backend.service.port        | Backend service port                               | `80`                                                                           |
| backend.service.annotations | Backend service annotations                        | Not Set                                                                        |
| backend.service.labels      | Backend service labels                             | Not Set                                                                        |

### Frontend

| Parameter                    | Description                                         | Default                                                                         |
|------------------------------|-----------------------------------------------------|---------------------------------------------------------------------------------|
| frontend.image.repository    | Frontend image repository                           | `europe-west1-docker.pkg.dev/woven-computing-234012/public/kubernetic-frontend` |
| frontend.image.tag           | Frontend image tag                                  | `3.1.0`                                                                         |
| frontend.image.pullPolicy    | Frontend image pull policy                          | `IfNotPresent`                                                                  |
| frontend.imagePullSecrets    | Frontend image pull secrets                         | `[]`                                                                            |
| frontend.resources           | Frontend resources allocation (Requests and Limits) | `{}`                                                                            |
| frontend.nodeSelector        | Frontend node labels for pod assignment             | `{}`                                                                            |
| frontend.tolerations         | Frontend toleration labels for pod assignment       | `[]`                                                                            |
| frontend.affinity            | Frontend affinity settings                          | `{}`                                                                            |
| frontend.annotations         | Frontend deployment annotations                     | Not Set                                                                         |
| frontend.labels              | Frontend deployment labels                          | Not Set                                                                         |
| frontend.podAnnotations      | Frontend pod annotations                            | Not Set                                                                         |
| frontend.podLabels           | Frontend pod labels                                 | Not Set                                                                         |
| frontend.service.type        | Frontend service type                               | `ClusterIP`                                                                     |
| frontend.service.port        | Frontend service port                               | `80`                                                                            |
| frontend.service.annotations | Frontend service annotations                        | Not Set                                                                         |
| frontend.service.labels      | Frontend service labels                             | Not Set                                                                         |

### Persistence

| Parameter                 | Description                            | Default         |
|---------------------------|----------------------------------------|-----------------|
| persistence.enabled       | Enable the use of a PVC                | `true`          |
| persistence.existingClaim | Provide the name of a pre-existing PVC | Not Set         |
| persistence.storageClass  | Storage class for the PVC              | Not Set         |
| persistence.accessMode    | The PVC access mode                    | `ReadWriteOnce` |
| persistence.size          | The size of the PVC                    | `1Gi`           |
| persistence.annotations   | Annotations for the PVC                | `{}`            |

### Configuration

| Parameter                    | Description                                                                                                               | Default        |
|------------------------------|---------------------------------------------------------------------------------------------------------------------------|----------------|
| config.log.level             | Log level (one of `info`, `debug`)                                                                                        | `info`         |
| config.in-cluster.enabled    | Flag to enable in-cluster kubernetes access through ServiceAccount                                                        | `true`         |
| config.auth.type             | authentication method (one of "token", "pomerium", "gke")                                                                 | Not Set        |
| config.auth.token.password   | password will be used for authentication of requests (used when `auth.type: token`)                                       | Not Set        |
| config.auth.pomerium.jwksURI | The attestation JWT's signature can be verified using the public key from this endpoint (used when `auth.type: pomerium`) | Not Set        |
| config.server.addr           | Listen to address                                                                                                         | `0.0.0.0:8080` |

### RBAC

| Parameter   | Description                        | Default |
|-------------|------------------------------------|---------|
| rbac.create | Whether RBAC resources are created | `true`  |

### ServiceAccount

| Parameter                  | Description                                          | Default                               |
|----------------------------|------------------------------------------------------|---------------------------------------|
| serviceAccount.create      | Specifies whether a ServiceAccount should be created | `true`                                |
| serviceAccount.name        | The name of the ServiceAccount to create             | Generated using the fullname template |
| serviceAccount.annotations | Specifies annotations to add to ServiceAccount       | Not Set                               |
