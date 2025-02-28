# gitops-playground

A repo for all sorts of GitOps experiments. It's a playground for me to try out different GitOps tools and techniques.

It is not and will not be production-ready. Please don't use it.

## Structure
```
common
└── addons # a source of cluster addons
    └── *.yaml

```

## ArgoCD ApplicationSet

To start using ApplicationSet from this repo, the following resources should be created in the cluster:

Cluster secret:
```yaml
apiVersion: v1
data:
  config: <redacted>
  name: <redacted>
  server: <redacted>
kind: Secret
metadata:
  annotations:
    common_repo_basepath: common
    common_repo_path: addons
    common_repo_revision: main
    common_repo_url: https://github.com/shini4i/gitops-playground.git
  labels:
    argocd.argoproj.io/secret-type: cluster
    cluster_name: disposable
    enable_reflector: "true" # enable the reflector controller
    enable_sealed_secrets: "true" # enable the sealed secrets controller
  name: in-cluster
  namespace: argo-cd
type: Opaque
```

App of Apps:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cluster-addons
  namespace: argo-cd
spec:
  destination:
    namespace: argo-cd
    server: https://kubernetes.default.svc
  project: default
  source:
    path: common/addons
    repoURL: https://github.com/shini4i/gitops-playground.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
