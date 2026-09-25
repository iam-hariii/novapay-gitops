# NovaPay GitOps

A Kubernetes and GitOps deployment setup for the NovaPay project using **Kind, Kubernetes, Helm, Argo CD, Git, and GitHub**.

The project demonstrates how Kubernetes application configuration can be stored in Git and synchronized to multiple environments through Argo CD.

## Architecture

```text
                    GitHub
              novapay-gitops
                     |
                     v
                  Argo CD
              /      |       \
             v       v        v
           Dev    Staging  Production
             |       |        |
           Nginx   Nginx    Nginx
```

Git is used as the desired-state source. Argo CD reconciles the Kubernetes cluster with the configuration stored in the repository.

## Technologies Used

- Docker Desktop
- Kind
- Kubernetes
- kubectl
- Helm
- Argo CD
- Git
- GitHub
- Nginx
- PowerShell

## Environments

| Environment | Kubernetes Namespace | Argo CD Application | Replicas Verified |
|---|---|---|---:|
| Development | `dev` | `nginx-dev` | 2 |
| Staging | `staging` | `nginx-staging` | 2 |
| Production | `production` | `nginx-production` | 4 |

## Repository Structure

```text
novapay-gitops/
├── apps/
│   ├── nginx/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── nginx-staging/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── nginx-production/
│       ├── deployment.yaml
│       └── service.yaml
│
└── argocd/
    ├── nginx-dev.yaml
    ├── nginx-staging.yaml
    └── nginx-production.yaml
```

## Kubernetes Setup

A Kind cluster named `novapay` was created and verified.

```text
kubectl get nodes

NAME                    STATUS   ROLES
novapay-control-plane   Ready    control-plane
```

Three namespaces were created:

```text
dev
staging
production
```

## Argo CD Setup

Argo CD was installed using Helm.

The Argo CD Applications use the GitHub repository as their source:

```text
https://github.com/iam-hariii/novapay-gitops.git
```

Each environment points to a separate repository path:

```text
Dev        -> apps/nginx
Staging    -> apps/nginx-staging
Production -> apps/nginx-production
```

Automated synchronization was enabled with:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### Configuration meaning

- **Automated sync**: Argo CD can synchronize desired Git state to Kubernetes.
- **Prune**: resources removed from the desired configuration can be removed from the cluster.
- **Self-heal**: live-state drift can be reconciled toward the Git-defined desired state.

## GitOps Demonstration

A production GitOps test was performed.

The production Deployment was initially configured with:

```yaml
replicas: 3
```

The value was changed in Git to:

```yaml
replicas: 4
```

The change was committed and pushed to GitHub:

```text
[main 8e82704] Scale production to four replicas
1 file changed, 1 insertion(+), 1 deletion(-)

9743108..8e82704  main -> main
```

After Argo CD reconciliation, Kubernetes reported:

```text
kubectl get deployment nginx -n production

NAME    READY   UP-TO-DATE   AVAILABLE
nginx   4/4     4            4
```

This demonstrated the GitOps flow:

```text
Git change
    |
    v
GitHub
    |
    v
Argo CD
    |
    v
Kubernetes
    |
    v
Production scaled from 3 to 4 replicas
```

No manual Kubernetes scaling command was used for this Git-driven change.

## Self-Healing / Reconciliation Tests

### Pod replacement

One Nginx pod was manually deleted from the production workload.

Kubernetes automatically created a replacement pod so that the Deployment continued to maintain its desired replica count.

### Live-state drift

A live Kubernetes Deployment was manually scaled away from its Git-defined desired state.

Argo CD's configured `selfHeal: true` policy reconciled the live state back toward the desired configuration.

## Sample Output

### Argo CD Applications

Final verification:

```text
NAME               SYNC STATUS   HEALTH STATUS
nginx-dev          Synced        Healthy
nginx-production   Synced        Healthy
nginx-staging      Synced        Healthy
```

### Production Deployment

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   4/4     4            4
```

### Production Pods

The final production verification showed four Nginx pods running successfully:

```text
nginx-...   1/1   Running
nginx-...   1/1   Running
nginx-...   1/1   Running
nginx-...   1/1   Running
```

## Final Result

The project successfully demonstrates:

- Local Kubernetes deployment using Kind
- Separate dev, staging, and production namespaces
- Argo CD installation and configuration
- GitHub-based GitOps repository
- Argo CD Applications for all three environments
- Automated synchronization
- Pruning enabled
- Self-healing enabled
- Git-driven production scaling
- Kubernetes pod replacement
- Live-state reconciliation
- Final Argo CD status of `Synced` and `Healthy` for all three environments

## Repository

GitHub:

https://github.com/iam-hariii/novapay-gitops
