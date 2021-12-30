# High-Availability

This will deploy 5 CentOS 7 virtual machines :
  * haproxy : Proxy  
  * kontrolplane1 : Control Plane
  * kontrolplane2 : Control Plane
  * kompute1 : Compute (node)  
  * kompute2 : Compute (node)  

You will need 10GB of RAM (2GB per hosts) for the cluster. You can ajust that in
the Vagrantfile (`domain.memory = 2048`).

## How to use

Clone this repository :
```
$ git clone https://github.com/OrioQuezac/k8s_deploy.git
```

Run `vagrant up` in this (`high-availability`) directory :
```
$ cd high-availability
$ vagrant up
```

Run the Ansible playbook with this command :
(the command is displayed in the output at the end of the `vagrant up` )
```
$ PYTHONUNBUFFERED=1 ANSIBLE_FORCE_COLOR=true ANSIBLE_HOST_KEY_CHECKING=false ANSIBLE_SSH_ARGS='-o UserKnownHostsFile=/dev/null -o IdentitiesOnly=yes -o ControlMaster=auto -o ControlPersist=60s' ansible-playbook --connection=ssh --timeout=30 --inventory-file=.vagrant/provisioners/ansible/inventory -v ansible/playbook.yml
```

Connect to HAProxy stats page with this link : http://10.0.0.10:9000/stats
This will show you all Control-Planes UP.

Check all created nodes (you will need kubectl on your host):
```
$ $ KUBECONFIG=admin.conf kubectl get nodes
NAME             STATUS   ROLES                  AGE     VERSION
kompute1         Ready    <none>                 43s     v1.23.1
kompute2         Ready    <none>                 43s     v1.23.1
kontrolplanes1   Ready    control-plane,master   2m19s   v1.23.1
kontrolplanes2   Ready    control-plane,master   102s    v1.23.1
```

or, connect to k1 :
```
$ vagrant ssh kontrolplanes1
Last login: Thu Dec 30 09:21:09 2021 from 192.168.121.1
[vagrant@kontrolplanes1 ~]$ sudo KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes
NAME             STATUS   ROLES                  AGE     VERSION
kompute1         Ready    <none>                 77s     v1.23.1
kompute2         Ready    <none>                 77s     v1.23.1
kontrolplanes1   Ready    control-plane,master   2m53s   v1.23.1
kontrolplanes2   Ready    control-plane,master   2m16s   v1.23.1
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
$ KUBECONFIG=admin.conf kubectl apply -k ../yaml-collection/k8s_dashboard
```

You can expose port's pod with :
```
$ KUBECONFIG=admin.conf kubectl expose pod $(KUBECONFIG=admin.conf kubectl get pod -n kubernetes-dashboard --no-headers -o custom-columns=":metadata.name" | grep kubernetes-dashboard) -n kubernetes-dashboard --type=NodePort --name=kubernetes-ui
```

Get the Nodeport :
```
$ KUBECONFIG=admin.conf kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
dashboard-metrics-scraper   ClusterIP   10.101.171.153   <none>        8000/TCP         6m18s
kubernetes-dashboard        ClusterIP   10.99.6.118      <none>        443/TCP          6m18s
kubernetes-ui               NodePort    10.96.172.247    <none>        8443:31707/TCP   6m12s
```

Connect to Dashboard with `https://10.0.0.10:<exposed-port>` (Here, exposed port is 31707)

Get token authentication :
```
$ KUBECONFIG=admin.conf kubectl -n kubernetes-dashboard get secret $(KUBECONFIG=admin.conf kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```
