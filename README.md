# Learning Kubernetes

Auto-K8s is an automation setup for [Kubernetes](https://kubernetes.io/) cluster environment using Vagrant and Virtualbox. Ansible is used to automate Kubernetes features. This is a minimal setup to facilitate learning, development and usage of Kubernetes platform for container cluster management.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development, learning and testing purposes.

### Prerequisites

You will require the following to be available in your local machine to be able to proceed with the project

- [Vagrant](https://www.vagrantup.com/downloads.html)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Git](https://git-scm.com/downloads)

Do not worry if you know nothing about the pre-requisites. You can follow
[Getting Started with Vagrant](https://www.vagrantup.com/intro/getting-started/) and [VirtualBox](https://www.virtualbox.org/wiki/Technical_documentation) to learn more about them.

Note: It is recommended to use latest version of Vagrant i.e. >= 1.8.7, as earlier versions had issues with Ansible 2.0. I used the following versions:

Vagrant version 1.8.7

VirtualBox Version 5.1.22

### Installation and setup

A step by step approach that will get you a Kubernetes env running.

1. First clone the project to your local machine.

```
>> git clone git@github.com:naikajah/auto-k8s.git
>> cd auto-k8s
```

2. You will now create Kubernetes master or controller node. Details about Kubernetes master/controller node is explained in detail [here](kubernetes/playbooks/controller/README.md).

```
>> vagrant up k8s-master
```

Wait for the master/controller to be up. Once the master/controller is active, verify that the following services are up and running. More about these services [here](kubernetes/playbooks/controller/README.md).

```
>> vagrant ssh k8s-master
>> sudo systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler | grep Active
```

All these services should be in a running state.

```
[root@kube-master vagrant]# systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler | grep Active
   Active: active (running) since Thu 2017-07-20 19:51:07 CEST; 1min 31s ago
   Active: active (running) since Thu 2017-07-20 19:51:07 CEST; 1min 30s ago
   Active: active (running) since Thu 2017-07-20 19:51:08 CEST; 1min 30s ago
   Active: active (running) since Thu 2017-07-20 19:51:08 CEST; 1min 30s ago
```

3. Now since we got the kubernetes master/controller healthy, wealthy and wise, its time to bring pretty minions up. More on minions [here](kubernetes/playbooks/minion/README.md).

Start a new terminal window for minions.

```
>> vagrant up k8s-minion1
```

Wait for the first minion to be up. Once the minion is active, verify that the following services are up and running. More about these services [here](kubernetes/playbooks/minion/README.md).

```
>> vagrant ssh k8s-minion1
>> sudo systemctl status kube-proxy kubelet docker | grep Active
```

All these services should be in a running state.

```
[root@kube-minion1 vagrant]# systemctl status kube-proxy kubelet docker | grep Active
   Active: active (running) since Thu 2017-07-20 19:55:07 CEST; 9min ago
   Active: active (running) since Thu 2017-07-20 20:04:42 CEST; 9min ago
   Active: active (running) since Thu 2017-07-20 19:55:09 CEST; 9min ago
```

Now the Kubernetes is configured and ready to be explored.

## Verifying the Kubernetes setup

ssh to k8s-master and execute the following to verify if the minion1 is registered to it

```
>> kubectl get nodes
```

If minion1 has registered itself properly with the master/controller then you should see the following output

```
[root@kube-master vagrant]# kubectl get nodes
NAME           STATUS    AGE
kube-minion1   Ready     10m
```

Similarly you can bring up k8s-minion2 and k8s-minion3 up depending on the resources available in your machine. You can also configure the memory and vcpu requirements in the nodes.yaml file [here](nodes.yaml).

WELL DONE!!!!!! You have successfully configured Kubernetes.

## Setting up your first POD

A POD in Kubernetes is a group of one or more containers.

As part of this automation a minimal POD yaml configuration is provided at /opt/development directory in master/controller server.

In order to start with POD, execute the below commands to get your first POD running

```
>> vagrant ssh k8s-master
>> cd /opt/development
>> [root@kube-master development]# kubectl create -f ./pod_nginx.yaml
pod "nginx" created

>> [root@kube-master development]# kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          46s
```

In order to verify our POD, execute the below command

```
>> kubectl describe pod nginx
```

This will show the  detail information about the POD and and its containers.

```
[root@kube-master development]# kubectl describe pod nginx
Name:		nginx
Namespace:	default
Node:		kube-minion1/192.168.1.25
Start Time:	Thu, 20 Jul 2017 20:31:43 +0200
Labels:		<none>
Status:		Running
IP:		172.17.0.2
Controllers:	<none>
Containers:
  nginx:
    Container ID:	docker://e256f15080b68476ebf4afe7006f5a562b2b82256c8f823bf5fb746208ed5416
    Image:		nginx:1.7.9
    Image ID:		docker-pullable://docker.io/nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    Port:		80/TCP
    State:		Running
      Started:		Thu, 20 Jul 2017 20:32:00 +0200
    Ready:		True
    Restart Count:	0
    Volume Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mvvl7 (ro)
    Environment Variables:	<none>
```

In order to see the running containers. Connect to the minion associated with the POD.

```
>> vagrant ssh k8s-minion1
[root@kube-minion1 vagrant]# docker ps
CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS               NAMES
e256f15080b6        nginx:1.7.9                                "nginx -g 'daemon off"   6 minutes ago       Up 6 minutes                            k8s_nginx.f80b328f_nginx_default_ae1256bf-6d79-11e7-a2cf-08002775ff5f_de324289
4c514a0ee6ea        gcr.io/google_containers/pause-amd64:3.0   "/pause"                 6 minutes ago       Up 6 minutes                            k8s_POD.b2390301_nginx_default_ae1256bf-6d79-11e7-a2cf-08002775ff5f_39470741
```

WELL DONE!!!! again.

## Authors

* **Piyush Kumar** - [Github](https://github.com/naikajah)

## License
