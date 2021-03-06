---
description: Setting up a Kubernetes Cluster with Hetzner Dedicated Servers
---

# Setup - Hetzner Bare Metal

> Checkout this [repository](https://github.com/delphidigital/bare-metal-cluster-manager) to manage a cluster of dedicated servers on Hetzner.

The scripts in this repository will setup and maintain one or more [kubernetes](https://kubernetes.io) clusters consisting of dedicated [Hetzner](https://www.hetzner.com) servers. Each cluster will also be provisioned to operate as a node in the [THORCHain](https://thorchain.org) network.

Executing the scripts in combination with some manual procedures will get you clusters with the following features on bare metal.

* [Kubespray](https://kubespray.io/) \(based\)
* Internal NVMe storage \([Ceph](https://ceph.io)/[Rook](https://rook.io)\)
* Virtual LAN \(also over multiple locations\) \([Calico](https://www.projectcalico.org)\)
* Load Balancing \([MetalLB](https://metallb.universe.tf)\)

## Preparations

### Servers

Acquire a couple of [servers](https://www.hetzner.com/dedicated-rootserver/matrix-ax) as the basis for a cluster \(`AX41-NVME`'s are working well for instance\). Visit the [admin panel](https://robot.your-server.de/server) and name the servers appropriately.

```text
tc-k8s-master
tc-k8s-worker1
tc-k8s-worker2
...
```

Refer to the [reset procedure](setup-hetzner-bare-metal.md#resetting-the-bare-metal-servers) to properly initialize them.

### vSwitch

Create a [vSwitch](https://robot.your-server.de/vswitch/index) and order an appropriate subnet \(it may take a while to show up after the order\). Give the vSwitch a name \(i.e. `tc-k8s-net`\) and Assign this vSwitch to the servers.

Checkout the [docs](https://docs.hetzner.com/robot/dedicated-server/network/vswitch) for help.

## Usage

Clone this repository, `cd` into it and download kubespray.

```bash
git submodule init && git submodule update
```

Create a Python virtual environment or similar.

```bash
# Optional
virtualenv -p python3 venv
```

Install dependencies required by Python and Ansible Glaxy.

```bash
pip install -r requirements.python.txt
ansible-galaxy install -r requirements.ansible.yml
```

> Note: Mitogen does not work with ansible collections and needs to be disabled.

### Provisioning

```bash
cp hosts.example inventory/inventory.ini
cp cluster.yml.example private-cluster.yml
```

Add your server ip's to `inventory.ini` and your network information into `private-cluster.yml`

If you want to manage multiple clusters simply name the files according to the pattern below.

```text
private-cluster-01.yml
private-cluster-02.yml
private-cluster-02.yml
...

private-test.yml
...

private-helsinki-01.yml
...

private-whatever.yml
```

```bash
# Manage a cluster
ansible-playbook private-cluster.yml

# If you want to run kubespray separately
ansible-playbook kubespray/cluster.yml
```

> Check [this](https://kubespray.io/) out for more playbooks on cluster management.

### THORChain

In order for the cluster to operate as a node in the THORCHain network deploy as instructed [here](https://docs.thorchain.org/thornodes/kubernetes/deploying). You can also refer to the [node-launcher repository](https://gitlab.com/thorchain/devops/node-launcher), if necessary, or the THORChain [documentation](https://docs.thorchain.org) as a whole.

## Resetting the bare metal servers

Visit the [console](https://robot.your-server.de/server) and put each server of the cluster into rescue mode. Then execute the following script.

```bash
installimage -a -r no -i images/Ubuntu-1804-bionic-64-minimal.tar.gz -p /:ext4:all -d nvme0n1 -f yes -t yes -n hostname
```

This will install and use Ubuntu on only one of the two internal NVMe drives. The unused ones will be used for persistent storage with ceph/rook. You can check the internal drive setup with `lsblk`. Change it accordingly in the command shown above when necessary.

> Ubuntu 18.04 is used because kubespray does not support 20.04 \(yet\)

