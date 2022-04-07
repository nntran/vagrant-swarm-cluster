# vagrant-swarm-cluster
A sample Docker Swarm cluster with Vagrant and Ansible

## Prerequisites

* [Virtualbox 6+](https://www.virtualbox.org/wiki/Downloads)

* [Vagrant 2.2.14+](https://www.vagrantup.com/downloads.html)
* Ansible 2.12.2+

## Step 1: Clone the repository

```
git clone https://github.com/nntran/vagrant-swarm-cluster.git
```

## Step 2: Set up your cluster configuration

Open the inventory file (`cluster.yml`) and adapt the configuration according to your private network and the number of worker.

By default, the cluster consists of 3 VMs (1 master and 2 worker).

```yaml
all:
  vars:
    ansible_connection: ssh
    ansible_user: vagrant
    ansible_ssh_pass: vagrant
    ansible_become: true
    ansible_become_user: root
    ansible_python_interpreter: /usr/bin/python3
  hosts:
    vm-swarm-1:
      ansible_host: 10.11.13.51
    vm-swarm-2:
      ansible_host: 10.11.13.52
    vm-swarm-3:
      ansible_host: 10.11.13.53
  children:
    master:
      hosts:
        vm-swarm-1:
      vars:
        node: "master"
    worker:
      hosts:
        vm-swarm-2:
        vm-swarm-3:
      vars:
        node: "worker"
```

## Step 3: Create the virtual machines (VM) with [Vagrant](https://www.vagrantup.com/)

```
vagrant up
```

Example of logs:

```
==> Cluster config: 
{"all"=>{"vars"=>{"ansible_connection"=>"ssh", "ansible_user"=>"vagrant", "ansible_ssh_pass"=>"vagrant", "ansible_become"=>true, "ansible_become_user"=>"root", "ansible_python_interpreter"=>"/usr/bin/python3"}, "hosts"=>{"vm-swarm-1"=>{"ansible_host"=>"10.11.13.51"}, "vm-swarm-2"=>{"ansible_host"=>"10.11.13.52"}, "vm-swarm-3"=>{"ansible_host"=>"10.11.13.53"}}, "children"=>{"master"=>{"hosts"=>{"vm-swarm-1"=>nil}, "vars"=>{"node"=>"master"}}, "worker"=>{"hosts"=>{"vm-swarm-2"=>nil, "vm-swarm-3"=>nil}, "vars"=>{"node"=>"worker"}}}}}
==> user/password: vagrant/vagrant
==> hosts: 
{"vm-swarm-1"=>{"ansible_host"=>"10.11.13.51"}, "vm-swarm-2"=>{"ansible_host"=>"10.11.13.52"}, "vm-swarm-3"=>{"ansible_host"=>"10.11.13.53"}}
==> host: ["vm-swarm-1", {"ansible_host"=>"10.11.13.51"}]
==> vm_name: vm-swarm-1
==> vm_ip: 10.11.13.51
==> host: ["vm-swarm-2", {"ansible_host"=>"10.11.13.52"}]
==> vm_name: vm-swarm-2
==> vm_ip: 10.11.13.52
==> host: ["vm-swarm-3", {"ansible_host"=>"10.11.13.53"}]
==> vm_name: vm-swarm-3
==> vm_ip: 10.11.13.53
Bringing machine 'vm-swarm-1' up with 'virtualbox' provider...
Bringing machine 'vm-swarm-2' up with 'virtualbox' provider...
Bringing machine 'vm-swarm-3' up with 'virtualbox' provider...
==> vm-swarm-1: Importing base box 'generic/ubuntu2004'...
Progress: 20%
...
```

By default, each VM is created with the following characteristics:

* **vcpu**: 2
* **ram**: 2 GB

You can change it in the `Vagrantfile`:

```ruby
# -*- mode: ruby -*-
# # vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version '>= 2.2.4'
VAGRANTFILE_API_VERSION = '2'

# Require YAML module
require 'yaml'

file_root = File.dirname(File.expand_path(__FILE__))

# Global vars
cluster_name = 'Cluster K8S demo'
provider = 'virtualbox'
vm_memory = '2048'
vm_cpus = '2'
vm_disk = 20,
box_name = 'generic/ubuntu2004'
box_version =  '3.0.28'
...
```

> Vagrant will ask you to specify the network interface if you are running on other OS than Mac OS.

After a few minutes, all the defined VM are created and operational. You will see it with Virtualbox.

![](docs/virtualbox.png)

## Step 4: Create the Docker Swarm cluster using `Ansible`

1. Check that you have access to your VMs via SSH

```
ansible-playbook -i cluster.yml ansible/playbooks/check.yml
```

```
vm-swarm-1                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vm-swarm-2                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vm-swarm-3                 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```


2. Update hosts and upgrade OS

```
ansible-playbook -i cluster.yml ansible/playbooks/update-host.yml
```

```
ansible-playbook -i cluster.yml ansible/playbooks/upgrade.yml
```

The last task will take a few minutes.

3. Create the Docker Swarm cluster

The following command will install common and docker packages first then initialize the Swarm cluster. 

```
ansible-playbook -i cluster.yml ansible/site.yml
```

4. Check your cluster status

You can execute the following command on the master (manager) node. 

```
docker node ls
```

![](docs/nodes-info.png)


## How to remove the cluster ?

The following command will remove the Docker Swarm cluster and Virtualbox VMs

```
vagrant destroy -f
```

