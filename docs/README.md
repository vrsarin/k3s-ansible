# Build a Kubernetes cluster using K3s via Ansible
Author: <https://github.com/itwars>  
Current Maintainer: <https://github.com/dereknola>

## Ansible Collection:
**Name**: vrsarin.k3s_ansible

Current Maintainer: <https://github.com/vrsarin>

## Requirements:
- Ansible >= 2.14.0
- Python >- 3.10

## Installation:
These modules are distributed as [collections](https://docs.ansible.com/ansible/latest/collections_guide/index.html). To install them, run:
```shell
ansible-galaxy collection install vrsarin.k3s_ansible
```
Alternatively put the collection into a requirements.yml file:

```yaml
---
collections:
- vrsarin.k3s_ansible

```

## Usage

This collection comprise of 4 roles. 

1. **k3s-prereq**: this role will prepare machines for installing k3s cluster
1. **k3s-server**: this role initiates the cluster or add server to existing cluster
1. **k3s-agent**: this role add an agent machine to existing cluster
1. **k3s-upgrade**: this role will upgrade the 

Create a inventory file as given below. Change values as per your needs

```yaml
k3s_cluster:
  children:
    server:
      hosts:
        192.16.35.11:
    agent:
      hosts:
        192.16.35.12:
        192.16.35.13:

  # Required Vars
  vars:
    ansible_port: 22
    ansible_user: debian
    k3s_version: v1.26.9+k3s1
    token: "mytoken"  # Use ansible vault if you want to keep it secret
    api_endpoint: "{{ hostvars[groups['server'][0]]['ansible_host'] | default(groups['server'][0]) }}"
    extra_server_args: ""
    extra_agent_args: ""

  # Optional vars
    cluster_context: k3s-ansible
    api_port: 6443
    k3s_server_location: /var/lib/rancher/k3s
    systemd_dir: /etc/systemd/system
    extra_service_envs: [ 'ENV_VAR1=VALUE1', 'ENV_VAR2=VALUE2' ]
    # Manifests or Airgap should be either full paths or relative to the playbook directory.
    # List of locally available manifests to apply to the cluster, useful for PVCs or Traefik modifications.
    extra_manifests: [ '/path/to/manifest1.yaml', '/path/to/manifest2.yaml' ]
    airgap_dir: /tmp/k3s-airgap-images
    user_kubectl: true, by default kubectl is symlinked and configured for use by ansible_user. Set to false to only kubectl via root user.
    server_config_yaml:  |
      This is now an inner yaml file. Maintain the indentation.
      YAML here will be placed as the content of /etc/rancher/k3s/config.yaml
      See https://docs.k3s.io/installation/configuration#configuration-file
    registries_config_yaml:  |
      Containerd can be configured to connect to private registries and use them to pull images as needed by the kubelet.
      YAML here will be placed as the content of /etc/rancher/k3s/registries.yaml
      See https://docs.k3s.io/installation/private-registry
```
### Deploy Cluster
Create playbook as shown below

``` yaml
---
- name: Cluster prep
  hosts: k3s_cluster
  gather_facts: true
  become: true
  roles:
    - role: prereq
    - role: airgap
    - role: raspberrypi

- name: Setup K3S server
  hosts: server
  become: true
  roles:
    - role: k3s_server

- name: Setup K3S agent
  hosts: agent
  become: true
  roles:
    - role: k3s_agent
```
Run playbook

```shell
ansible-playbook playbook.yml -i inventory.yml
```
