# jetlag

Tooling to install clusters for testing via an on-prem [Assisted Installer](https://github.com/openshift/assisted-installer) on the Scale lab or Alias lab hardware.

Three separate layouts of clusters can be deployed:

* BM - Baremetal - 3 control-plane nodes, X number of worker nodes
* RWN - Remote Worker Node - 3 control-plane/worker nodes, X number of remote worker nodes
* SNO - Single Node OpenShift - 1 OpenShift Master/Worker Node "cluster" per available hardware resource

Each cluster layout requires a bastion machine which is the first machine out of your lab "cloud" allocation. The bastion machine will host the assisted-installer and serve as a router for remote worker node clusters. BM and RWN layouts produce a single cluster consisting of 3 control-plane nodes and X number of worker or remote worker nodes. SNO layout creates an SNO cluster per available machine after fufilling the bastion machine requirement.

## Ansible Prerequisites

Pre-reqs for the playbooks:

```console
$ ansible-galaxy collection install ansible.posix
$ ansible-galaxy collection install containers.podman
$ ansible-galaxy collection install community.general
```

## Cluster Deployment Usage

Edit vars

```console
$ cp ansible/vars/all.sample.yml ansible/vars/all.yml
$ vi ansible/vars/all.yml
```

Set your pull-secret in `pull_secret.txt` in repo base directory.

Make sure to set/review the following vars:

* `lab_cloud`, `lab_quads_host`, `lab_foreman_host`, `lab_dns` - for lab related details
* `cluster_type` - either `bm`, `rwn`, or `sno` for the respective cluster layout
* `worker_node_count` - applies to bm and rwn cluster types for the desired worker count, ideal for leaving left over inventory hosts for other purposes
* `controlplane_lab_interface` - applies to bm and rwn cluster types and should map to the nodes interface in which the lab provides dhcp to
* `controlplane_network` and `controlplane_network_*` - applies to bm and rwn cluster types, for bm it will be the network that each node, api, and ingress addresses are assigned from, for rwn it is only the 3 control-plane nodes, api and ingress addresses
* `rwn_lab_interface` - applies only to rwn cluster type and should map to the nodes interface in which the lab provides dhcp to
* `rwn_vlan_start` - desired starting vlan for remote worker nodes
* `rwn_network_*` - network configuration to be used for each remote worker node, each remote worker will be on its own network

Run create-inventory playbook

```console
ansible-playbook ansible/create-inventory.yml
```

Run setup-bastion playbook

```console
ansible-playbook -i ansible/inventory/cloud42.local ansible/setup-bastion.yml
```

Run deploy for either bm/rwn/sno playbook with inventory created by create-inventory playbook

Bare Metal Cluster:

```console
ansible-playbook -i ansible/inventory/cloud42.local ansible/bm-deploy.yml
```

Remote Worker Node Cluster:

```console
ansible-playbook -i ansible/inventory/cloud42.local ansible/rwn-deploy.yml
```

Single Node OpenShift:

```console
ansible-playbook -i ansible/inventory/cloud42.local ansible/sno-deploy.yml
```

## Workload Usage

Review README.md in [workload](workload) directory.
