# GitOps (WIP)

## Introduction

https://www.weave.works/blog/what-is-gitops-really

* GIT as the source of truth for the cluster
* GIT as is the single place where we operate (create, change, destroy) for all environments
* Changes are all observable via GIT

## Tools Used

* [ArgoCD](https://github.com/argoproj/argo-cd)
* [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
* [Ambassador](https://ambassador.io)
* [Kustomize](https://github.com/kubernetes-sigs/kustomize)

## Installation

### 1. Prerequisites

A kubernetes cluster with `kubectl` configured. More to come here. I recommend [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), the k8s that comes with docker, or [k3s](https://k3s.io). There is also numerous cloud options.

### 2. Install Sealed Secrets

[Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) to allow secrets to be declared openly in this repository. Sealed secrets are encrypted using public/private key encryption (AES-256). The private key resides within the cluster and is used to decrypt Sealed Secrets to Secrets.

#### 2.1 Add Sealed Secrets to the cluster

```sh
kubectl -f apply cluster/base/sealedsecrets.yaml
```

#### 2.2 Set the Sealed Secrets private key

When initially installing Sealed Secrets in a cluster, the Sealed Secrets controller will generate a private key. 

When a new cluster is created for a new infrastructure environment you must use the same private key to allow existing secrets to be unsealed in that cluster. You should back this key up and store in an encrypted file store.

```sh
# Replace the master key
kubectl delete -f master.key
kubectl apply -f master.key

# Delete the controller pod (this will cause it to restart and read the new key)
kubectl delete pod -l name=sealed-secrets-controller
```

#### 3. Apply the kustomization

Sanity check the cluster config with:

```sh
kustomize build cluster/env/prod
```

If the kustomization looks good, apply with:

```sh
kustomize build cluster/env/prod/ | kubectl apply -f -
```

The combined kustomization will:

* Create cluster namespaces
* Install Ambassador
* Install ArgoCD and create ArgoCD apps that will sync this new configuration to the cluster

#### 4. Ingress/network config

Currently you will need to configure you ingress and such for ambassador. When this is no longer WIP you will see an example of doing this delcaratively using `external-dns` and `cert-manager`. You will need a cluster on the internet though, without more work you would not be able to do this on your local.

## Usage

Once the cluster is setup and Argo is deployed, you can give GitOps a shot by upgrading the redis version.

```sh
git checkout -b upgrade-redis
```

Change `.images[{name=='redis'}].newTag` to the version of redis you want to upgrade to. Commit and review the PR. On merge, Argo will see the change and redeploy the redis deployment. You could use `yq` to do this as well.

## References
- https://blog.argoproj.io/the-state-of-kubernetes-configuration-management-d8b06c1205
- https://www.socallinuxexpo.org/sites/default/files/presentations/Get%20Started%20with%20GitOps.pdf
- https://www.youtube.com/watch?v=uvbaxC1Dexc&list=PLj6h78yzYM2PpmMAnvpvsnR4c27wJePh3&index=30&t=0s
- https://www.weave.works/blog/what-is-gitops-really
- https://www.weave.works/blog/gitops-high-velocity-cicd-for-kubernetes
- https://www.twistlock.com/2018/08/06/gitops-101-gitops-use/
- https://medium.com/faun/monorepo-for-gitops-5402fe7a8e35
- https://thenextweb.com/contributors/2018/12/08/all-you-need-to-know-about-gitops/
- https://www.weave.works/blog/gitops-is-cloud-native
- https://github.com/stefanprodan/gitops-istio
- https://github.com/bricef/gitops-tutorial
- https://www.weave.works/blog/kubernetes-anti-patterns-let-s-do-gitops-not-ciops
- https://www.slideshare.net/weaveworks/continuous-lifecycle-london-2018-event-keynote-97418556

## More Useful Tools
- [KubePlatform](https://kubeplatform.dev/)
- [yq](https://github.com/kislyuk/yq)
- [yamllint](https://github.com/adrienverge/yamllint)
- https://ramitsurana.github.io/awesome-kubernetes/
