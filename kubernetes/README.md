# Why Ansible?

I chose Ansible purely for the fact that I have very little or rather no experience in it and I wanted to learn a new platform besides puppet and chef (which I already have some experience with) for automation. Unlike Puppet or Chef it doesn’t use an agent on the remote host. Instead Ansible uses SSH which is assumed to be installed on all the systems you want to manage.

*Note* This might not be the best code I have written and will definitely evolve over time as I learn more Ansible concepts and tricks.

# Ansible usage

## Directory sturcture

The directory structure followed in the project is as below:

```
kubernetes
├── README.md
├── inventories
│   ├── development
│   │   ├── group_vars
│   │   │   ├── all
│   │   │   ├── controllers.yaml
│   │   │   └── minions.yaml
│   │   ├── host_vars
│   │   └── inventory_dev
│   ├── preproduction
│   └── production
├── playbooks
│   ├── controller
│   │   ├── README.md
│   │   └── playbook.yaml
│   └── minion
│       ├── README.md
│       └── playbook.yaml
└── roles
   ├── controller
   │   ├── files
   │   │   └── pod_nginx.yaml
   │   ├── tasks
   │   │   └── main.yaml
   │   └── templates
   │       ├── apiserver.j2
   │       ├── controller-manager.j2
   │       └── etcd.conf.j2
   ├── kubernetes
   │   ├── files
   │   │   ├── hosts
   │   │   └── virt7-common-release.repo
   │   ├── tasks
   │   │   └── main.yaml
   │   └── templates
   │       └── config.j2
   ├── linux
   │   └── tasks
   │       └── main.yaml
   └── minion
       ├── tasks
       │   └── main.yaml
       └── templates
           └── kubelet.j2
```

## Nodes

 - [nodes.yaml] (nodes.yaml): The file contains the details of the vagrant virtual machines that can be provisioned via this project.

The format of the file is shown below:

```
< nodename >:
  box: < boxname >
  server:
    hostname: < hostname of the box >
    memory: < amount of memory allocated to the VM >
    vcpu: < virtual CPU allocated to the VM >
    ip: < IP address can be provided >
  provisioning:
    role: < Ansible role served by the VM >
    environment: < environment that is being created >
```

## Inventories

The inventory file located in `inventories/development/inventory_dev` (this is the inventory file for development environment) contains the host entries and groups that they belong to. e.g.

```
[controllers]
k8s-master ansible_connection=local

[minions]
k8s-minion1 ansible_connection=local
```

This is the standard group format from Ansible. This ensures that `k8s-master` node belongs to the `controllers` group.

The groups are created to ensure same configuration across the similar set of nodes. More about Ansible inventories [here](http://docs.ansible.com/ansible/intro_inventory.html).

*NOTE* Make sure the nodenames are same across the nodes.yaml and inventory file.

## Playbooks

Playbooks can be considered as a group containing one or more roles.

At present there are 2 playbooks in the project each containing various roles.
- [controller](playbooks/controller/playbook.yaml)
- [minion](playbooks/minion/playbook.yaml)

## Roles

Roles define the configuration of the virtual machine being provisioned. A machine can have more than one role associated with it via playbooks as described in the section above.

Following roles are present as of today:

- [controller](roles/controllers): (Configuration specific to the k8s master/controller)
- [minion](roles/minion): (Configuration specific to the k8s minion/node)
- [kubernetes](roles/kubernetes): (Configuration that is common to master and minion)
- [linux](roles/linux): (OS specific configuration)
