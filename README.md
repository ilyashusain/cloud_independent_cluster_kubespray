# cloud_independent_cluster_kubespray

## Prerequisites:

- Any hypervisor that can spin up Ubuntu machines

## Introduction:

In this article we will create a kubernetes cluster from former principles. This will be achieved in a cloud independent fashion using the open-source deployment tool *kubespray*. Additionally, we will deploy a Rancher dashboard and advanced logging tooling on to our cluster.

This will be done on google cloud, a high-level cloud; this is for the sake of simplicity. However, as will be demonstarted, the cloud provider is not important, and therefore this guide is applicable to all cloud providers. All that matters is that the machines exist and are up.

Kubespray operates by way of ansible playbooks (an open-source automation tool written in python) that provision the necessary components onto the machines, thereby qualifying them as *nodes*. So in the case of kubespray, there is a pre-existing common automation framework (Ansible) that does the heavy work for us and is conveniently human readable, too.

# Brief:

In the steps that follow, we will create 3 Ubuntu 18.04 instances comprising of 1 bastion host, 1 master, 1 worker machine (for the sake of simplicity and time), provision the machines with the necessary kubernetes components qualifying them as nodes, install Rancher for dashboarding, and install helm to deploy an EFK stack (elasticsearch, fluentbit and kibana) for advanced logging functionality and visualization.

## Guide:

1. Create 3 ubuntu 18.04 instances with at least 2 vCPU. 1 instance is a bastion host, 1 instance is a master, 1 instance is a worker. You can do this manually, or via an infrastucture-as-code automation tool.

*NOTE:* If you go below 2 vCPU on your nodes, you will not be able to run the fluentbit and elasticsearch pods. These are generally cpu and memory hungry applications given the rate at which data is being queried and its scope.

2. ssh into the bastion host, in the home directory run (copy and paste into the shell):

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

4. On the master node, execute `sudo vim /etc/sudoers` and copy this line in with your username:

`<ENTER USERNAME HERE> ALL=(ALL) NOPASSWD:ALL`

save with :wq!. Repeat this step on the worker node.

5. Download kubespray, install the requirments file and create an ansible inventory for your cluster by executing:

```git clone https://github.com/kubernetes-sigs/kubespray.git && cd kubespray && sudo pip install -r requirements.txt && cp -rfp inventory/sample inventory/mycluster```

6. Declare the ips to be used by ansible, these should be the internal ips of the vms:

`declare -a IPS=(<MASTER NODE INTERNAL IP> <WORKER NODE INTERNAL IP>)`

and set the config file environemnt variable:

`CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}`

7. Deploy the cluster:

`ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml`

8. ssh to your master node and run:

`mkdir -p $HOME/.kube && sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && sudo chown $(id -u):$(id -g) $HOME/.kube/config`

You can now interact with your cluster from the master.

9. On the bastion host, install docker and then install rancher:

```
sudo apt install docker.io -y
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

We do not want to deploy the rancher container on to the cluster, as it can easily churn which is troublesome if no persistent volumes are attached.

Now you can access the rancher web UI from the bastion hosts IP entered into a browser, then create an admin account and login. Import your cluster into Rancher by following the instructions. After a few minutes, you will have a clean dashboard that can visualize and edit the components within your kubernetes cluster.

10. Install helm on the master node and deploy the server side component (tiller) on to your cluster:

```
sudo snap install helm --classic
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

You can now deploy the EFK helm chart on to your cluster, be it from the command line or by plain deployments via the rancher UI Apps (<cluster name>/Default, select "Apps" from the top panel, then click "Launch", scroll down until you see the EFK stack and click "view details", scroll down again and click "Launch"), this decision is up to you as the result is the same.
