# K3D Rancher Setup

## What is this?
This repo contains a basic structure to allow you to easily roll K3D setups, with or without Rancher.

## What do I need?
First, you need a few tools installed. You can do that any way you like, but I prefer to use [Homebrew](https://brew.sh) on my Mac. Install the following:

- k3d
- kubectl
- helm (v3)
- realpath (this is usually part of the `coreutils` package)
- Docker Desktop (or a native Docker daemon if you're on Linux)

## How do I create stuff?
Without configuring anything else, you can get a simple K3D setup without Rancher by running the `k3d-rancher-setup` script: 

```bash
± ./k3d-rancher-setup

K3D Setup
=========
The following values have been configured:

Cluster Name                  : mycluster
# of Server Nodes             : 1
# of Agent Nodes              : 3
Cluster Agent Volume Path     : ./agent-volume
Configure Registry in Cluster : 0
Configure Rancher in Cluster  : 0
Kubeconfig Path               : ./kubeconfig/mycluster

Do you want to create a K3D cluster with the settings above? (y/N)
```

Hit `y` on your keyboard, and within a minute or two you'll have a running K3D cluster.

## What can I configure?
You can set a number of envvars to configure the K3D cluster to your liking:

- `K3D_CLUSTER_NAME`: name your cluster (default: mycluster)
- `K3D_CLUSTER_SERVERS`: how many control plane nodes do you want? (default: 1)
- `K3D_CLUSTER_AGENTS`: how many agents do you want? (default: 3)
- `K3D_AGENT_VOLUME`: you can bind-mount a local path into each agent
- `K3D_SETUP_REGISTRY`: do you want a private registry? Set this to `1` (default: 0)
- `K3D_SETUP_RANCHER`: do you want Rancher? Set this to `1` (default: 0)
- `KUBECONFIG_DIR`: where do you want to store the custom KUBECONFIG files? (default: ./kubeconfig)

For example:

```bash
± K3D_SETUP_REGISTRY=1 K3D_SETUP_RANCHER=1 ./k3d-rancher-setup

K3D Setup
=========
The following values have been configured:

Cluster Name                  : mycluster
# of Server Nodes             : 1
# of Agent Nodes              : 3
Cluster Agent Volume Path     : ./agent-volume
Configure Registry in Cluster : 1
Configure Rancher in Cluster  : 1
Kubeconfig Path               : ./kubeconfig/mycluster

Do you want to create a K3D cluster with the settings above? (y/N) y
WARN[0000] No node filter specified
INFO[0000] Prep: Network
INFO[0000] Re-using existing network 'k3d-mycluster' (fa1fd0a1b86a56bd0ebf02a523356fb142ca665294752c2e41c83df4a895853e)
INFO[0000] Created volume 'k3d-mycluster-images'
INFO[0000] Creating node 'k3d-mycluster-registry'
INFO[0000] Successfully created registry 'k3d-mycluster-registry'
INFO[0001] Creating node 'k3d-mycluster-server-0'
INFO[0001] Creating node 'k3d-mycluster-agent-0'
INFO[0001] Creating node 'k3d-mycluster-agent-1'
INFO[0001] Creating node 'k3d-mycluster-agent-2'
INFO[0001] Creating LoadBalancer 'k3d-mycluster-serverlb'
INFO[0001] Starting cluster 'mycluster'

...
```

