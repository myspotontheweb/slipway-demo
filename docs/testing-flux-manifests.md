# Testing Flux manifests

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