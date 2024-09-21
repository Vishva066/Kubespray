# KUBESPRAY

## Overview

Kubespray is an open-source tool that allows for the automated deployment of Kubernetes clusters across computing resources, such as virtual machines. 

Kubespray performs specific operations against those resources. For instance, it sets up the OS and installs packages, downloads and runs Container Runtime Interfaces (Containerd or Docker), creates infrastructure for secure connections between Kubernetes components, and more.

## How does Kubespray Work?

Kubespray utilizes a combination of Ansible and Kubeadm to deploy a Kubernetes cluster. 


## Ansible

Ansible is an open source, command-line IT automation software application written in Python.

## TUTORIAL

For Kubespray to work it needs ssh access in all the VMs and permission to run the command without being a sudo user.

**Note: Run the kubespray commands apart from the cluster node as it cannot ssh into its own node. Also set up authentication with username and password and not with SSH alone while creating VMs.**

Create a key-pair and copy it to all the VMs

```bash
ssh-keygen -t rsa
```

Copy to all the nodes in the cluster both master and worker

```bash
ssh-copy-id root@<ip>
```

If your user is not root then you have to set permission for that particualr user to be able run super user commands without the sudo command. 

For that execute this command in all the nodes

```bash
sudo nano /etc/sudoers
```

Enter your username in this placeholder

```bash
<username> ALL=(ALL) NOPASSWD:ALL
```

---

To install python execute this command:

```bash
sudo apt install python3-pip
```

Now clone the kubespray repsitory

```bash
git clone https://github.com/kubernetes-sigs/kubespray.git
```
---

Now Install Ansible. Here we are installing ansible in a virtual environment as suggested by the official documentation. As it keeps all the downloads in a seperate environment which does not affect other parts of the system.

We will also follow the same method. Also it will also work if not installed in a virtual environment.

Follow these commands to install ansible and other pre-requisites

```bash
VENVDIR=kubespray-venv

KUBESPRAYDIR=kubespray

python3 -m venv $VENVDIR

source $VENVDIR/bin/activate

cd $KUBESPRAYDIR

pip install -U -r requirements.txt
```

If you are unable to create a virtual environment then you may have a missing python version so intall this version

```bash
sudo apt install python3.10-venv
```

Then execute the above commands.

Copy the sample folder to your own folder here I have named the folder mycluster.

```bash
cp -rfp inventory/sample inventory/mycluster
```

Install the missing dependency

```bash
pip3 install ruamel.yaml
```
This library is used for parsing, editing, and writing YAML files in Python.

Define the IPs of all the nodes. Starting from master to worker.

```bash
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
```

Now add the IP in the hosts.yaml file

```bash
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

Review the hosts.yaml file

```bash
cd inventory/mycluster

nano hosts.yaml
```

In the hosts.yaml file:

- hosts - mention al the nodes including master and worker

- kube_control_pane section - Mention all the master nodes

- kube_node - enter the worker nodes 

- etcd hosts section - Enter the master nodes

Add the changes to the name also for eg if you want to call the node w1 or worker1. You can mention it there.

---

Now go back to the initial kubespray repo

```bash
cd ../..
```

Execute this ansible playbook command

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

The --become flag in Ansible is used to run tasks with elevated privileges, typically root or another specified user. It’s similar to using sudo on Linux when you need administrative rights to perform certain actions.

The option --become is required, as for example writing SSL keys in /etc/,
installing packages and interacting with various systemd daemons.
Without --become the playbook will fail to run!

**If you are given permission to do super user commands without the help of sudo in the become tag you can specify the username instead of root**

---

### Scaling the kubernetes cluster

Go to the hosts.yaml file and make changes accordingly based on worker or master node. 

Then execute this commadn

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root scale.yml --limit=worker4
```

Here the --limit=worker4 ensures that the scale.yml playbook is only applied to the worker4 node, instead of running it on all nodes listed in your inventory.

---

### Removing a kubernetes node

To remove a particular kubernetes node execute this command

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root remove-node.yml -e node=worker4
```

Here worker4 represents the node that we want to remove.

---

### To remove the cluster use this command

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml
```

## Resources

1. https://www.youtube.com/watch?v=zJoK8MMI23Q

2. https://www.youtube.com/watch?v=1W4w2ziRU8Q&t=508s

3.  https://github.com/kubernetes-sigs/kubespray