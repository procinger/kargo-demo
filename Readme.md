# Kargo.io Demo App

This Github repository demonstrates the Kargo.io controller. This repository focuses entirely on Kargo.io and will 
not mention the other controllers/operators. If you have any questions about these, please consult the relevant
documentation and help.

To run this demo successfully, the following components are required
- [Kubernetes](https://kubernetes.io/)
- [Argo CD](https://argoproj.github.io/cd/)
- [Argo Rollouts](https://argoproj.github.io/rollouts/)
- [Cert Manager](https://cert-manager.io/)
- [Kargo.io](https://kargo.io/)


### The content of the repository consists of
- A [Github Action](./.github/workflows/build-image.yml) to build and push an [image](./image)
- An [ArgoCD ApplicationSet](./argo-cd) which renders three environments
- The [Kargo.io Kubernetes manifests](./kargo) to create a project including workflows
- Our [demo app in Kubernetes Manifests](./kustomize), ready to deploy

### Rollout of the manifests
*For a better experience, this repository should be forked and all repository URLs in the Kubernetes manifests should
be replaced with your fork*

```
grep -iR "kargo-demo.git"
argo-cd/kargo-demo-application-set.yaml:        repoURL: https://github.com/procinger/kargo-demo.git
argo-cd/kargo-demo-application-set.yaml:        repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-test.yaml:                          repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-test.yaml:                          - repoURL: https://github.com/procinger/kargo-demo.git
kargo/warehouse.yaml:                           repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-prod.yaml:                          repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-prod.yaml:                          repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-prod.yaml:                          repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-prod.yaml:                          - repoURL: https://github.com/procinger/kargo-demo.git
kargo/git-creds.yaml:                           repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-dev.yaml:                           repoURL: https://github.com/procinger/kargo-demo.git
kargo/stage-dev.yaml:                           - repoURL: https://github.com/procinger/kargo-demo.git


grep -iR "ghcr.io"       
.github/workflows/build-image.yml:              tags: ghcr.io/procinger/kargo-demo:1.${{ github.run_number }}.${{ github.run_attempt }}
kustomize/base/deployment-kargo-demo.yaml:      image: ghcr.io/procinger/kargo-demo:1.10.1
kargo/stage-test.yaml:                          - image: ghcr.io/procinger/kargo-demo
kargo/warehouse.yaml:                           repoURL: ghcr.io/procinger/kargo-demo
kargo/stage-prod.yaml:                          - image: ghcr.io/procinger/kargo-demo
kargo/git-creds.yaml:                           repoURL: ghcr.io/procinger/kargo-demo
kargo/stage-dev.yaml:                           - image: ghcr.io/procinger/kargo-demo


```



1) Deploy the ArgoCD ApplicationSet in the ArgoCD Namespace
```
kubectl --namespace argo-cd apply -f argo-cd/kargo-demo-application-set.yaml
```

2) Create the Kargo.io Resources
```
kubectl create namespace kargo-demo
kubect --namespace kargo-demo apply -f kargp/*
```
3) Create and deploy a Github Personal Access Token and give Kargo.io the necessary Permissions `repo` & `write:packages` 
```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: kargo-demo-repo-git
  namespace: kargo-demo
  labels:
    kargo.akuity.io/cred-type: git
stringData:
  repoURL: https://github.com/procinger/kargo-demo.git
  username: <YOUR_USERNAME>
  password: <GITHUB_PAT>
```

After a successful deployment, two freights should have been created. A warehouse has this subscribed and the 
promotion chain is from dev > test > prod

It is important to note here that dev simulates a flaky environment and has a [random fail](./kargo/analysis-random-fail.yaml) 
in addition to an http [smoke test](./kargo/analysis-http-smoke-test.yaml). This [Argo rollout analysis template](https://argo-rollouts.readthedocs.io/en/stable/analysis/job/) acts like a coin flip.

![kargo-demo](https://github.com/user-attachments/assets/da97a810-9187-4cf0-9c5a-a26aea47e2c2)

