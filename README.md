# slipway-demo

A repository that demonstrates the use of the slipway project. 

Slipway is a tool which can generate your GitOps configuration and is ideal for new projects

## Getting started

### Setup Templates

Configure the [Slipway project](https://github.com/myspotontheweb/slipway) as a submodule

```bash
git submodule add https://github.com/myspotontheweb/slipway.git libs/slipway
```

Configure environment variable pointing at template location

```bash
export HYGEN_TMPLS=$(git rev-parse --show-toplevel)/libs/slipway/_templates
```

### Generate GitOps Manifests

```bash
#
# Generate
#
```

### Prepare cluster secrets

TODO

### Launch a cluster

TODO

### Cleanup

TODO