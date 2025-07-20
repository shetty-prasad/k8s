#### ETCD

 It is a distributed, reliable key-value store that is both simple and fast.

* Install etcd
 ```
wget -q --https-only "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
```

* examine the keys stored in etcd
```
kubectl exec etcd-master -n kube-system -- etcdctl get / --prefix --keys-only
```

* To access etcd in k8s use the etcdctl, you need to  explicitly set the etcd version to 3 which is the latest.
```
export ETCDCTL_API=3
```
```
kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / \
  --prefix --keys-only --limit=10 / \
  --cacert /etc/kubernetes/pki/etcd/ca.crt \
  --cert /etc/kubernetes/pki/etcd/server.crt \
  --key /etc/kubernetes/pki/etcd/server.key"
```
