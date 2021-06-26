# Install jenkins on kubernetes
### Prerequisites
- Helm
- Dynamic NFS

### Create namespace
```sh
$ kubectl create namespace jenkins
```

### Provision Dynamic NFS
Follow instructions here https://github.com/tixsalvador/provision-dynamic-nfs
```sh
$ kubectl create -f https://raw.githubusercontent.com/tixsalvador/provision-dynamic-nfs/main/rbac.yaml
$ kubectl create -f https://raw.githubusercontent.com/tixsalvador/provision-dynamic-nfs/main/class.yaml
$ kubectl create -f https://raw.githubusercontent.com/tixsalvador/provision-dynamic-nfs/main/deployment.yaml
```

#### Configure Helm
``` sh
$ helm repo add jenkinsci https://charts.jenkins.io
$ helm repo update
$ helm search repo jenkinsci
```

********** NOT YET VERIFIED FROM THIS LINE *****************
#### Configure jenkins config files
```sh
$ helm inspect values jenkinsci/jenkins  > jenkins.values.yaml

# Edit jenkins.values.yaml
Look for:
serviceType: ClusterIP
storageClass:
Change to:
serviceType: NodePort
storageClass: managed-nfs-storage

# Install helm using jenkins.values.yaml
$ helm install jenkins jenkinsci/jenkins --values jenkins.values.yaml  --namespace jenkins
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
