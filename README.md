# Containerized Ansible Environment with Podman

This repository contains all required artifacts and instructions to setup several Podman containers to experiement with Ansible.

By following the instructions below, you will:

* setup a Podman machine to run containers from
* setup an Ansible controller container from which playbooks can be executed from
* setup 3 test hosts that will be used as target hosts in your playbook
* test connectivity between the Ansible controller container and the 3 test host containers
* run a playbook that applies several different types of plays to some of the test host containers
* clean up containers and images when your done (optional)

All container images are based on Red Hat Universal Base Image (UBI) version 9 and do not RHEL subscriptions. UBI images are essentially simplified, open-source versions of Red Hat Enterprise Linux.

## Prerequisites

Required software:

* Windows Sub-System for Linux (WSL2)
* Podman for Windows

## Instructions

### Setup Podman Containers

To setup Ansible contoller and test host containers, run these commands in bash shell from the root of this repo:

```bash 
# generate SSH keys for Ansible to connect between containers
ssh-keygen -q -f images/.ssh/id_rsa -N ''

# create, start and connect to podman machine
podman machine init --rootful=true
podman machine start
wsl -d podman-machine-default

# build images
podman build --tag ansible-controller -f /mnt/d/workspaces/ansible/images/ansible-controller.yml
podman build --tag ansible-test-host -f /mnt/d/workspaces/ansible/images/ansible-test-host.yml
podman image ls

# create 1 Ansible controller container and 3 test containers
podman network create ansible-network
podman run -dti --name ansible-controller --network ansible-network -v /d/workspaces/ansible:/ansible localhost/ansible-controller
podman run -dti -p 2221:22 --network ansible-network --name ansible-test-host-1 localhost/ansible-test-host
podman run -dti -p 2222:22 --network ansible-network --name ansible-test-host-2 localhost/ansible-test-host
podman run -dti -p 2223:22 --network ansible-network --name ansible-test-host-3 localhost/ansible-test-host
podman container ls
```

### Run Ansible playbook

Connect to the Ansible controller, test connectivity to test hosts and run your first playbook:

```bash
# connect to Ansible controller, test connections and run playbook
podman attach ansible-controller
ansible group1 -m ping -i /ansible/inventory.ini
ansible group2 -m ping -i /ansible/inventory.ini
ansible-playbook /ansible/playbook.yml -i /ansible/inventory.yml

# disconnect from Ansible controller with key sqeuence: <Ctrl-P> <Ctrl-Q>
```

### Clean up

Forcefully removing each image will also stop and remove containers associated to those images:

```bash
podman image rm localhost/ansible-controller -f
podman image rm localhost/ansible-test-host -f
```
