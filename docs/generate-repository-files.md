# Generate Repository Files

When Generating files make sure the following environment variable is set

```bash
export HYGEN_TMPLS=libs/slipway/_templates
```

## Setup Templates

Configure the [Slipway project](https://github.com/myspotontheweb/slipway) as a submodule

```bash
git submodule add https://github.com/myspotontheweb/slipway.git libs/slipway
```

## Generate Brewfile

```bash
hygen slipway Brewfile
```

### Generate GitOps Manifests

```bash
export CLUSTER=dev1

rm -rf flux # optional

hygen slipway controllers
hygen slipway demo-configs
hygen slipway cluster --name $CLUSTER
hygen slipway eksctl --name $CLUSTER --region eu-west-1
```

### Secret Management (SOPS)

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
