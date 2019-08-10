# cloud_independent_cluster_kubespray

## Prerequisites:

- A hypervisor of your choice to spin up Ubuntu machines

## Brief:

In this article we will create a cluster comprising of 1 master, 1 node, 1 bastion host (for the sake of simplicity and time). This will be achieved in a cloud independent fashion using the open-source deployment tool *kubespray*.

This will be done on google cloud, a high-level cloud; this too is for the sake of simplicity. However, as will be demonstarted, the cloud provider is not important, and therefore this guide is applicable to all cloud providers. All that matters is that the ubuntu machines exist and are up.

Kubespray operates by way of ansible playbooks (an open-source automation tool written in python) that provision the necessary components onto the machines, thereafter qualifying them as *nodes*. So in the case of kubespray, there is a pre-existing common framework that integrates itself effortlessly into the machines hardware; kubespray does not need to know who or what is providing the hypervisor for the virtual machines soon-to-be nodes.

## Guide:

1. Create 3 ubuntu 18.04 instances. 1 instance is a bastion host, 1 instance is a master, 1 instance is a worker.

2. ssh into the bastion host, in the home directory run:

```
sudo apt update
sudo apt install python3-pip
sudo pip3 install --upgrade pip
cd .ssh && ssh-keygen -t rsa && cd
```

`cat` the `id_rsa.pub` file and copy this to your clipboard.

3. ssh into the master node and edit the .ssh/authorized_keys file, copy in the public key you just copied.

4. On the master node, execute `sudo vim /etc/sudoers` and copy in with your username:

`<ENTER USERNAME HERE> ALL=(ALL) NOPASSWD:ALL`

save with :wq!.
