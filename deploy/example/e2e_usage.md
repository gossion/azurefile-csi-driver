## CSI driver E2E usage example
#### 1. create a pod with csi azurefile driver mount on linux
##### Option#1: Azurefile Dynamic Provisioning
 - Create an azurefile CSI storage class
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/storageclass-azurefile-csi.yaml
```

 - Create an azurefile CSI PVC
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/pvc-azurefile-csi.yaml
```

##### Option#2: Azurefile Static Provisioning(use an existing azure file share)
 - Use `kubectl create secret` to create `azure-secret` with existing storage account name and key
```
kubectl create secret generic azure-secret --from-literal accountname=NAME --from-literal accountkey="KEY" --type=Opaque
```

 - Create an azurefile CSI PV, download `pv-azurefile-csi.yaml` file and edit `shareName` in `volumeAttributes`
```
wget https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/pv-azurefile-csi.yaml
vi pv-azurefile-csi.yaml
kubectl create -f pv-azurefile-csi.yaml
```

 - Create an azurefile CSI PVC which would be bound to the above PV
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/pvc-azurefile-csi-static.yaml
```

#### 2. validate PVC status and create an nginx pod
 - make sure pvc is created and in `Bound` status finally
```
watch kubectl describe pvc pvc-azurefile
```

 - create a pod with azurefile CSI PVC
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/nginx-pod-azurefile.yaml
```

#### 3. enter the pod container to do validation
 - watch the status of pod until its Status changed from `Pending` to `Running` and then enter the pod container
```
$ watch kubectl describe po nginx-azurefile
$ kubectl exec -it nginx-azurefile -- bash
root@nginx-azurefile:/# df -h
Filesystem                                                                Size  Used Avail Use% Mounted on
overlay                                                                   30G   19G  11G   65%  /
tmpfs                                                                     3.5G  0    3.5G  0%   /dev
...
//f571xxx.file.core.windows.net/pvc-54caa11f-9e27-11e9-ba7b-0601775d3b69  1.0G  64K  1.0G  1%   /mnt/azurefile
...
```
In the above example, there is a `/mnt/azurefile` directory mounted as cifs filesystem.
