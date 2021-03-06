# k8s_deploy

Vagrantfile, scripts and a yaml collection to deploy a k8s cluster.

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

## How to use

Clone this repository :
```
$ git clone https://github.com/OrioQuezac/k8s_deploy.git
```

Run `vagrant up` in k8s_deploy directory :
```
$ cd k8s_deploy
vagrant up
```

Check all created pods (you will need kubectl on your host):
```
$ KUBECONFIG=admin.conf kubectl get pods --all-namespaces
NAMESPACE     NAME                         READY   STATUS    RESTARTS   AGE
kube-system   coredns-64897985d-kkk4b      1/1     Running   0          10m
kube-system   coredns-64897985d-rz77s      1/1     Running   0          10m
kube-system   etcd-k1                      1/1     Running   0          10m
kube-system   kube-apiserver-k1            1/1     Running   0          10m
kube-system   kube-controller-manager-k1   1/1     Running   0          10m
kube-system   kube-flannel-ds-27ql7        1/1     Running   0          10m
kube-system   kube-flannel-ds-b9zgf        1/1     Running   0          10m
kube-system   kube-flannel-ds-znqmz        1/1     Running   0          10m
kube-system   kube-proxy-jqsk7             1/1     Running   0          10m
kube-system   kube-proxy-mml2p             1/1     Running   0          10m
kube-system   kube-proxy-rvmq2             1/1     Running   0          10m
kube-system   kube-scheduler-k1            1/1     Running   0          10m
```

or, connect to k1 :
```
$ vagrant ssh k1
[vagrant@k1 ~]$ sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get pods --all-namespaces
NAMESPACE     NAME                         READY   STATUS    RESTARTS   AGE
kube-system   coredns-64897985d-kkk4b      1/1     Running   0          13m
kube-system   coredns-64897985d-rz77s      1/1     Running   0          13m
kube-system   etcd-k1                      1/1     Running   0          13m
kube-system   kube-apiserver-k1            1/1     Running   0          13m
kube-system   kube-controller-manager-k1   1/1     Running   0          13m
kube-system   kube-flannel-ds-27ql7        1/1     Running   0          13m
kube-system   kube-flannel-ds-b9zgf        1/1     Running   0          13m
kube-system   kube-flannel-ds-znqmz        1/1     Running   0          13m
kube-system   kube-proxy-jqsk7             1/1     Running   0          13m
kube-system   kube-proxy-mml2p             1/1     Running   0          13m
kube-system   kube-proxy-rvmq2             1/1     Running   0          13m
kube-system   kube-scheduler-k1            1/1     Running   0          13m
```

**Enjoy !**

## Add Storage

Apply a StorageClass with yaml-collection/sc-slow.yaml :
```
$ KUBECONFIG=admin.conf kubectl apply -f yaml-collection/sc-slow.yaml
```

Create directories for PersistentVolumes on each node :
```
$ for i in {1..3};do vagrant ssh -c 'sudo mkdir /mnt/data{1..6}' k$i; done
```

Add PersistentVolumes with the template in yaml-collection/pv-data.yaml
```
$ KUBECONFIG=admin.conf kubectl apply -f yaml-collection/pv-data.yaml
```

Check if PersistentVolumes are available :
```
$ KUBECONFIG=admin.conf kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
data1-storage   10Gi       RWO            Delete           Available           slow                    10s
data2-storage   10Gi       RWO            Delete           Available           slow                    10s
data3-storage   10Gi       RWO            Delete           Available           slow                    10s
data4-storage   10Gi       RWO            Delete           Available           slow                    10s
data5-storage   10Gi       RWO            Delete           Available           slow                    10s
data6-storage   10Gi       RWO            Delete           Available           slow                    10s
```

## Deploy Kubernetes Dashboard

To deploy Kubernetes Dashboard :
```
$ KUBECONFIG=admin.conf kubectl apply -k yaml-collection/k8s_dashboard
```

You can expose port's pod with :
```
$ KUBECONFIG=admin.conf kubectl expose pod $(kubectl get pod -n kubernetes-dashboard --no-headers -o custom-columns=":metadata.name" | grep kubernetes-dashboard) -n kubernetes-dashboard --type=NodePort --name=kubernetes-ui
```

Get `k1` ip :
```
$ vagrant ssh -c "ip a | grep -A 3 eth0:" k1
```

Connect to Dashboard with `https://<ip-k1>:<exposed-port>`

Get token authentication :
```
$ KUBECONFIG=admin.conf kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```
