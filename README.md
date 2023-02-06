# Ansible/Helm Playbooks

These are a series of Ansible Playbooks that use [Helm](https://helm.sh) to install packages to Kubernetes clusters.  These playbooks are refactorings of the [CloudmanBoot](https://github.com/CloudVE/cloudman-boot.git) playbooks and are intended to be used to install the various components in isolation. 

These playbooks should also be refactored into proper Ansible roles so they can be easily used in other projects.

## Requirements

These playbooks use the Ansible Galaxy `kubernetes.core` collection which **must** be installed before using the playbooks.

```bash
ansible-galaxy collection install kubernetes.core
```

## Inventories

The `inventories` directory contains a Jinja2 template that can be used as a starting point to create your own inventory, or it can be rendered with a script like [render_template.py](https://gist.github.com/ksuderman/82d3d3f91df794439f46842d2dcc07ab) to generate an inventory:



The `render_template.py` script (should you choose to use it) takes one required parameter (`-t|--template`) and zero or more key/value pairs that will be used as parameter values in tbe template.  The `inventory.ini.j2` template expects three required values for the controller node and zero or more worker definitions.

1. **name** the hostname of the controller node.  This template only supports a single controller node.
1. **ip** the IP address of the controller node.
1. **key** the SSH key used to connect to the node. 
```
render_template.py -t inventories/inventory.ini.j2 key=~/.ssh/key.pem name=master ip=1.2.3.4
```

For a multi-node cluster set the worker names and IP addresses in the `workers` parameter:

```
render_template.py -t inventories/inventory.ini.js key=~/.ssh/key.pem name=master ip=1.2.3.4 workers="worker1:1.2.3.5 worker21.2.3.6"
```

## Playbooks

1. **setup.yml**<br/>Installs Python, Pip, and other OS dependencies (i.e. nfs-common, OpenShift)
1.  **rke/main.yml**<br/>Installs RKE2 and configure the controller and all worker nodes, if any.
1. **secrets.yml**<br/>Sets the `cloud-config` secret in the K8S cluster from the value of the `KUBE_CLOUD_CONF` environment variable.
1. **helm.yml**<br/>Installs Helm and sets the *stable* and *cloudve* repositories.
1. **storage.yml**<br/>Installs CSI drivers.  Currently only OpenStack (Cinder CSI) is supported.
1. **nfs.yml**<br/>Installs the Ganesha NFS provisioner.
1. **cert-manager.yml**<br/>Installs Cert Manager
1. **ingress.yml**<br/>Installs the Nginx ingress controller.
1. **galaxy.yml**<br/>Installs Galaxy.
1. **rancher.yml**<br/>Installs the Rancher application.

## Caveats and known issues

These playbooks have really only been used on Jetstream and Jetstream2 (OpenStack) and the conditionals for AWS and GCP have been removed (actually not copied) from the original playbooks.