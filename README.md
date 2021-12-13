# k8s_deploy

Vagrantfile and scripts to deploy a k8s cluster.

**Requirements :**
* libvirt
* nfsd
* Vagrant
  * vagrant-libvirt plugin

This will deploy 3 CentOS 7 virtual machines :
  * k1 : Control Pane
  * k2 : Compute (node)
  * k3 : Compute (node)

You will need 6GB of RAM (2GB per hosts) for the cluster. You can ajust that in
the Vagrantfile (`domain.memory = 2048`).

## How to use

Clone this repository :
```
git clone https://github.com/OrioQuezac/k8s_deploy.git
```

Run `vagrant up` in k8s_deploy directory :
```
cd k8s_deploy
vagrant up
```

Enjoy !
