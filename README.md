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















*
