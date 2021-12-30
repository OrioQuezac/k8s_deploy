# k8s_deploy

Vagrantfile, scripts and a yaml collection to deploy a k8s cluster.

**Requirements :**
* libvirt
* nfsd
* Vagrant
  * vagrant-libvirt plugin


## Simple-Cluster directory

This will deploy 3 CentOS 7 virtual machines :
  * k1 : Control Pane
  * k2 : Compute (node)
  * k3 : Compute (node)

You will need 6GB of RAM (2GB per hosts) for the cluster. You can ajust that in
the Vagrantfile (`domain.memory = 2048`).

Go to the README in `simple-cluster` directory for some instructions.

## High-Availability directory

This will deploy 5 CentOS 7 virtual machines :
  * haproxy : Proxy  
  * kontrolplane1 : Control Plane
  * kontrolplane2 : Control Plane
  * kompute1 : Compute (node)  
  * kompute2 : Compute (node)  

The sizing of VMs is the same as `simple-cluster` (2GB per hosts).

You can configure the amount of VMs with the `N` variable in Vagrantfile (default is `2`).

Go to the README in `high-availability` directory for some instructions.

## How to use

Clone this repository :
```
$ git clone https://github.com/OrioQuezac/k8s_deploy.git
```

Run `vagrant up` in `simple-cluster` or `high-availability` directory :
```
$ cd simple-cluster
$ vagrant up
```
