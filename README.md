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

### Create Persistent Volume Claim
```sh
# Edit jenkins.pvc-nfs.yaml with the following values
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-pvc
  namespace: jenkins
spec:
  storageClassName: managed-nfs-storage
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
##################
$ kubectl create -f jenkins-pvc-nfs.yaml

## To check ###
$ kubectl get pods,pv,pvc -A
```

### Create and deploy  Jenkins deployment file
```sh
$ kubectl create -f jenkins-deployment.yaml
```

### Grant access to Jenkins service
```sh
# Edit jenkins-service.yaml with ff values
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: jenkins
######################
$ kubectl apply -f jenkins-service.yaml
```

e0222f083ded47a59040edcabbd2c76a













********** NOT YET VERIFIED FROM THIS LINE *****************
#### Configure Helm
``` sh
$ helm repo add jenkinsci https://charts.jenkins.io
$ helm repo update
$ helm search repo jenkinsci
```

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
