# kube-etcd-controller

Managed etcd clusters on Kubernetes:

- creation
- destroy
- resize
- recovery
- backup
- rolling upgrade

## Requirements

- Kubernetes 1.4+
- etcd 3.0+

## Limitations

- Backup only works for data in etcd3 storage, not etcd2 storage.

## Initialize the TPR

(TODO: auto create TPR when deploy the controller)

```bash
$ cat example/etcd-clusters-tpr.yaml
```

```yaml
apiVersion: extensions/v1beta1
kind: ThirdPartyResource
description: "Managed etcd clusters"
metadata:
  name: "etcd-cluster.coreos.com"
versions:
  - name: v1
  - name: v2
```

```bash
$ kubectl create -f example/etcd-clusters-tpr.yaml

$ kubectl get thirdpartyresources
NAME                      DESCRIPTION             VERSION(S)
etcd-cluster.coreos.com   Managed etcd clusters   v1,v2
```


## Deploy kube-etcd-controller

```bash
$ kubectl create -f example/etcd-controller.yaml
pod "kubeetcdctrl" created
```

## Create an etcd cluster

```bash
$ cat example/example-etcd-cluster.yaml
```

```yaml
apiVersion: "coreos.com/v1"
kind: "EtcdCluster"
metadata:
  name: "example-etcd-cluster"
size: 3
```

```bash
$ kubectl create -f example/example-etcd-cluster.yaml
```

```bash
$ kubectl get pods
NAME                READY     STATUS    RESTARTS   AGE
etcd-cluster-0000   1/1       Running   0          11s
etcd-cluster-0001   1/1       Running   0          11s
etcd-cluster-0002   1/1       Running   0          11s
```

```bash
$ kubectl get services
NAME                CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
etcd-cluster-0000   10.0.104.18    <none>        2380/TCP,2379/TCP   45s
etcd-cluster-0001   10.0.243.108   <none>        2380/TCP,2379/TCP   45s
etcd-cluster-0002   10.0.45.68     <none>        2380/TCP,2379/TCP   45s
kubernetes          10.0.0.1       <none>        443/TCP             8m
```

```bash
$ kubectl log etcd-cluster-0000
...
2016-08-05 00:33:32.453768 I | api: enabled capabilities for version 3.0
2016-08-05 00:33:32.454178 N | etcdmain: serving insecure client requests on 0.0.0.0:2379, this is strongly discouraged!
```

## Destroy an existing etcd cluster

```bash
$ kubectl delete -f example/example-etcd-cluster.yaml
```

```bash
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
```
## Try cluster recovery

Simulate a pod failure by simply delete it

```bash
$ kubectl delete pod etcd-cluster-0000
```

etcd controller will recover the failure by creating a new pod `etcd-cluster-0003`

```bash
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
etcd-cluster-0001   1/1       Running   0          5s
etcd-cluster-0002   1/1       Running   0          5s
etcd-cluster-0003   1/1       Running   0          5s
```