# Install jenkins on kubernetes
### Prerequisites
- Helm
- kubernetes
-Dynamic NFS 

### Create namespace
```sh
$ kubectl create namespace jenkins
```

### Create Dynamic NFS
Follow instructions here https://github.com/tixsalvador/provision-dynamic-nfs

### Install jenkins using Helm

#### Configure Helm
``` sh
$ helm repo add jenkinsci https://charts.jenkins.io
$ helm repo update
$ helm search repo jenkinsci
```

#### Create a persistent volume
Create a file named jenkins-volume.yaml with content
```sh
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  namespace: jenkins
spec:
  storageClassName: jenkins-pv
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 20Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/nfs/data/jenkins-volume
```

Apply the spec
```sh
$ kubectl apply -f jenkins-volume.yaml
$ kubectl describe pv jenkins-pv -n jenkins
```
