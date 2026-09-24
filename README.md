# slipway-demo

A repository that demonstrates the use of the slipway project. 

Slipway is a tool which can generate your GitOps configuration and is ideal for new projects

## Getting started

### Setup Templates

Configure the [Slipway project](https://github.com/myspotontheweb/slipway) as a submodule

```bash
git submodule add https://github.com/myspotontheweb/slipway.git libs/slipway
```

### Install software

```bash
#
# Setup hygen
#
brew tap jondot/tap
brew install hygen
softwareupdate --install-rosetta --agree-to-license

#
# Install the rest using a Brewfile bundle
#
hygen slipway Brewfile
brew bundle install
```

### Generate GitOps Manifests

```bash
export CLUSTER=dev1
export HYGEN_TMPLS=libs/slipway/_templates

rm -rf flux # optional

hygen slipway controllers
hygen slipway demo-configs
hygen slipway cluster --name $CLUSTER
hygen slipway eksctl --name $CLUSTER --region eu-west-1
```

### Prepare cluster secrets

```bash
export CLUSTER=dev1
export POS=0

#
# Generate a fresh age key
#
rm *.agekey *.agekey.pub
age-keygen -o age.agekey
age-keygen -y age.agekey > age.agekey.pub

#
# Create Sops configuration file
#
echo "" > .sops.yaml
yq -i '.creation_rules[env(POS)].path_regex = "flux/infrastructure/core/"+strenv(CLUSTER)+"/.*\.sops\.yaml$"' .sops.yaml
yq -i '.creation_rules[env(POS)].encrypted_regex = "^(data|stringData)$"' .sops.yaml
yq -i '.creation_rules[env(POS)].key_groups[0].age[0] += load("age.agekey.pub")' .sops.yaml
yq -i '.creation_rules[env(POS)].key_groups[0].age[1] += load(strenv(HOME) + "/.ssh/id_ed25519.pub")' .sops.yaml

#
# Save the flux decryption secret
#
kubectl create secret generic sops-age -n flux-system --from-file age.agekey --dry-run=client -oyaml > flux/infrastructure/core/$CLUSTER/sops-age.sops.yaml
sops -e -i flux/infrastructure/core/$CLUSTER/sops-age.sops.yaml
```

### Launch a cluster

```bash
eksctl create cluster -f eks/config/dev1.yaml
```

Bootstrap FluxCD

```bash
export CONTEXT=eksctl
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

## Cleanup

```bash
eksctl delete cluster -f eks/config/dev1.yaml
```

## Testing

You can review the manifest generation as follows

```bash
export CLUSTER=dev1

# infra-core
flux build kustomization infra-core \
  --kustomization-file flux/clusters/$CLUSTER/00-core.yaml \
  --path $(yq '.spec.path' flux/clusters/$CLUSTER/00-core.yaml) \
  --dry-run

# infra-controllers
flux build kustomization infra-controllers \
  --kustomization-file flux/clusters/$CLUSTER/01-controllers.yaml \
  --path $(yq '.spec.path' flux/clusters/$CLUSTER/01-controllers.yaml) \
  --dry-run

# infra-configs
flux build kustomization infra-configs \
  --kustomization-file flux/clusters/$CLUSTER/02-configs.yaml \
  --path $(yq '.spec.path' flux/clusters/$CLUSTER/02-configs.yaml) \
  --dry-run
```