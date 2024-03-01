# Containerized Ansible Environment with Podman

This repository contains all required artifacts and instructions to setup several Podman containers to experiement with Ansible.

Follow the instructions below to:

* setup a Podman machine to run containers
* setup an Ansible controller container from which you can experiement with and execute playbooks
* setup 3 test host containers that are included in your Ansible inventory and serve as targets for your playbooks
* test connectivity between the Ansible controller container and the 3 test host containers
* run a playbook that applies several different types of plays to the test host containers
* optionally clean up all containers and images when your done

All the container images are based on [Red Hat Universal Base Image (UBI)](https://catalog.redhat.com/software/base-images) version 9 and do not require RHEL subscriptions or licenses. Red Hat UBIs are essentially simplified, open-source, freely available versions of Red Hat Enterprise Linux.

## Prerequisites

Required software:

* Windows Sub-System for Linux (WSL2)
* Podman for Windows

## Instructions

### Setup Podman Containers

To setup Ansible contoller and test host containers, run these commands in bash shell from the root of this repo:

```bash 
# generate SSH keys for Ansible to connect between containers
mkdir ./Containerfiles/.ssh
ssh-keygen -q -f ./Containerfiles/.ssh/id_rsa -N ''

# create, start and connect to podman machine
podman machine init --rootful=true
podman machine start
wsl -d podman-machine-default

# build images
podman build --tag ansible-controller -f ./Containerfiles/ansible-controller.yml
podman build --tag ansible-test-host -f ./Containerfiles/ansible-test-host.yml
podman image ls

# create 1 Ansible controller container and 3 test containers
podman network create ansible-network
podman run -dti --name ansible-controller --network ansible-network -v ./:/ansible localhost/ansible-controller
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
