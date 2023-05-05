# Astria Dev Cluster

This repository contains configuration and related scripts for running Astria with Podman.

## Dependencies

Main dependencies

* Rust - https://www.rust-lang.org/tools/install
* Podman - https://podman.io/getting-started/installation

For contract deployment:

* Forge (part of Foundry) - https://book.getfoundry.sh/getting-started/installation

## Setup

```bash
# init and start podman machine
podman machine init
podman machine start

# create template yaml
# NOTE - replace the geth_local_account with your own account address
podman run --rm \
  -e pod_name=astria_stack \
  -e celestia_home_volume=celestia-home-vol \
  -e metro_home_volume=metro-home-vol \
  -e executor_home_volume=executor-home-vol \
  -e relayer_home_volume=relayer-home-vol \
  -e conductor_home_volume=conductor-home-vol \
  -e executor_local_account=0xb0E31D878F49Ec0403A25944d6B1aE1bf05D17E1 \
  -e celestia_app_host_port=26657 \
  -e bridge_host_port=26659 \
  -e sequencer_host_port=1318 \
  -e sequencer_host_grpc_port=9100 \
  -e executor_host_http_port=8545 \
  -e executor_host_grpc_port=50051 \
  -e scripts_host_volume="$PWD"/container-scripts \
  -v "$PWD"/templates:/data/templates \
  dcagatay/j2cli:latest \
  -o /data/templates/astria_stack.yaml \
  /data/templates/astria_stack.yaml.jinja2

# run pod
podman play kube --log-level=debug templates/astria_stack.yaml

# deploy test contract
git clone https://github.com/joroshiba/test-contracts
cd test-contracts
export PRIV_KEY=123...
RUST_LOG=debug forge create --private-key $PRIV_KEY src/Storage.sol:Storage

```

### Helpful commands

```bash
# list running containers
podman ps

# stop running stack
podman pod stop astria_stack

# remove stack
podman pod rm astria_stack
```

### Helpful links

* https://podman.io/getting-started/
* https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
* https://kubernetes.io/docs/concepts/configuration/configmap/
* https://www.redhat.com/sysadmin/podman-play-kube-updates
* https://www.redhat.com/sysadmin/podman-mac-machine-architecture
