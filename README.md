# K3D Rancher Setup

## What is this?
This repo contains a basic structure to allow you to easily roll K3D setups, with or without Rancher. This setup is heavily inspired by work from [Jason Yee](https://github.com/jwsy), but modified for my own needs.

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
Cluster API port              : 6550
Load Balancer HTTP port       : 80
Load Balancer HTTPS port      : 443
Kubeconfig Path               : ./kubeconfig/mycluster

Do you want to create a K3D cluster with the settings above? (y/N)
```

Hit `y` on your keyboard, and within a minute or two you'll have a running K3D cluster.

## What can I configure?
You can set a number of envvars to configure the K3D cluster to your liking:

- `K3D_CLUSTER_NAME`: name your cluster (default: mycluster)
- `K3D_K3S_IMAGE`: Docker image to use for K3s (default: `rancher/k3s`)
- `K3D_K3S_VERSION`: Docker image tag to use (default: `latest`)
- `K3D_CLUSTER_SERVERS`: how many control plane nodes do you want? (default: 1)
- `K3D_CLUSTER_AGENTS`: how many agents do you want? (default: 3)
- `K3D_AGENT_VOLUME`: you can bind-mount a local path into each agent
- `K3D_DNS_FORWARDERS`: optional list of DNS servers to use in Pods (default: undefined, which makes K3s use 8.8.8.8)
- `K3D_SETUP_REGISTRY`: do you want a private registry? Set this to `1` (default: 0)
- `K3D_SETUP_RANCHER`: do you want Rancher? Set this to `1` (default: 0)
- `K3D_LB_HTTP_PORT`: configure which local port to bind to the load balancer for HTTP (default: 80)
- `K3D_LB_HTTPS_PORT`: configure which local port to bind to the load balancer for HTTPS (default: 443)
- `KUBECONFIG_DIR`: where do you want to store the custom KUBECONFIG files? (default: ./kubeconfig)
- `K3D_HANDSFREE`: don't want a confirmation prompt? Set this to `1` (default: unset)

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
Cluster API port              : 6551
Load Balancer HTTP port       : 81
Load Balancer HTTPS port      : 444
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

## Port Collision Detection
If you try to run multiple clusters simultaneously, the `k3d-rancher-setup` script will automatically detect port collisions for the API, HTTP and HTTPS ports, and find the next free ports. This means that even if you set `K3D_LB_HTTP_PORT` you may end up with a different port. Check the output carefully!

## Custom DNS Servers
K3D runs a Docker-in-Docker setup, and uses its own Docker Bridge Network. Non-default Docker Bridge Networks use an internal DNS resolver that is a bit of magic on 127.0.0.11.

As a result, the K3D 'nodes' get an /etc/resolv.conf that points to an IP address that is not routable from a container. By default K3D works around that by having a 'backup config' that just hard-codes 8.8.8.8 (Google DNS) which may or may not be allowed on your company network. By configuring your own DNS forwarders you can override this backup config, and supply our own DNS servers instead.

Generally, if you need to access specific endpoints on your private/company network (not resolvable via 8.8.8.8) or if cluster creation gets stuck (for example on installing the Nginx Ingress controller Helm chart), this may be the option you need.

Example:
```
$ K3D_CLUSTER_NAME=mycluster K3D_DNS_FORWARDERS="10.1.1.11, 10.1.1.12, 8.8.8.8" ./k3d-rancher-setup

K3D Setup
=========
The following values have been configured:

Cluster Name                  : mycluster
K3s Image                     : rancher/k3s
K3s Version                   : latest
# of Server Nodes             : 1
# of Agent Nodes              : 3
Cluster Agent Volume Path     : ./agent-volume
Configure Registry in Cluster : 0
Configure Rancher in Cluster  : 0
Cluster API port              : 6551
Load Balancer HTTP port       : 81
Load Balancer HTTPS port      : 444
Custom DNS Forwarders         : 10.1.1.11, 10.1.1.12, 8.8.8.8
Kubeconfig Path               : ./kubeconfig/mycluster
```

## What are these extra directories for?
The extra directories `agent-volume` and `hack` are ignored by git by default. The `agent-volume` dir is the default directory to get bind-mounted into K3D agents for storage, and the `hack` dir gives you a nice place to put some K8s manifests while you're hacking around.

