# slipway-demo

A repository that demonstrates the use of the slipway project. 

Slipway is a tool which can generate your GitOps configuration and is ideal for new projects

## Docs

* [Generating repository files](docs/generate-repository-files.md)
* [Testing Flux manifests](docs/testing-flux-manifests.md)

## Getting started

### Install software

Using [Homebrew](https://brew.sh/)

```bash
brew bundle install
```

### Launch a cluster

Start a local cluster for testing

```bash
colima start --profile local-dev --kubernetes --kubernetes-version v1.37.0+k3s1 --cpus 4 --memory 8 --network-address
```

Bootstrap FluxCD

```bash
export CONTEXT=colima-local-dev
export CLUSTER=dev1
export GIT_OWNER=myspotontheweb
export GIT_REPO=slipway-demo
export GIT_BRANCH=main
export GITHUB_TOKEN=$(gh auth token)

# Flux secret
kubectl create ns flux-system --context $CONTEXT
sops -d flux/infrastructure/core/$CLUSTER/sops-age.sops.yaml | kubectl apply -f - --context $CONTEXT

# Bootstrap
flux bootstrap github --context $CONTEXT --owner=$GIT_OWNER --repository=$GIT_REPO --branch=$GIT_BRANCH --path=flux/clusters/$CLUSTER
```

### Cleanup

```bash
colima delete local-dev
```