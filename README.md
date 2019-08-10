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
sudo apt install python3-pip -y
sudo pip3 install --upgrade pip
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
git clone https://github.com/kubernetes-sigs/kubespray.git && cd kubespray && sudo pip install -r requirements.txt && cp -rfp inventory/sample inventory/mycluster
cd && cd .ssh && ssh-keygen -t rsa && cd
```

`cat` the `id_rsa.pub` file and copy this to your clipboard.

3. ssh into the master node and edit the .ssh/authorized_keys file, copy in the public key you just copied. Repeat this step for the worker node.

4. On the master node, execute `sudo vim /etc/sudoers` and copy in with your username:

`<ENTER USERNAME HERE> ALL=(ALL) NOPASSWD:ALL`

save with :wq!. Repeat this step on the worker node.

5. Download kubespray and install the requirments file:

git clone https://github.com/kubernetes-sigs/kubespray.git && cd kubespray && sudo pip install -r requirements.txt && cp -rfp inventory/sample inventory/mycluster

6. Declare the ips to be used by ansible, these should be the internal ips of the vms:

`declare -a IPS=(<MASTER NODE INTERNAL IP> <WORKER NODE INTERNAL IP>)`

and set the config file environemnt variable:

`CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}`

7. Deploy the cluster:

`ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml`

8. ssh to your aster node and run:

`mkdir -p $HOME/.kube && sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && sudo chown $(id -u):$(id -g) $HOME/.kube/config`

You can now interact with your cluster.
