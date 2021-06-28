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

### Create Jenkins Service Account
```sh
$ kubectl create -f jenkins-sa.yaml
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

### JENKINS UI
Once connected to jenkins UI

##### Configure kubernetes credential.
Dashboard -> Manage Jenkins -> Manage Credentials -> Jenkins -> Global Credentials
Add Credentials
Kind: Kubernetes Service Account
Scope: Global

##### Master Node Usage #####
(Optional only if exist)
To prevent master node from doing builds. Change usage policy

Dashboard -> Manage Jenkins -> Manage Nodes and Clouds -> master -> Configure
Usage: Only build jobs with label expression matching this node

##### Configure Kubernetes Cloud
Dashboard -> Manage Jenkins -> Manage Nodes and Clouds -> Configure Clouds -> Add new Cloud

Config changes  
Name: kubernetes  
Kubernetes URL: $(kubectl config view --minify | grep server | cut -f 2- -d ":" | tr -d " ")  
Kubernetes server certificate key: $(kubectl get secret $(kubectl get sa jenkins -n jenkins -o jsonpath={.secrets[
0].name}) -n jenkins -o jsonpath={.data.'ca\.crt'} | base64 --decode)  
Kubernetes Namespace: jenkins  
Credential: Secret text  
Websocket Enable (with check)  
Jenkins URL: http://10.10.10.10:32235  
Pod label:  
  Key: jenkins  
  Value: slave  
Pod Templates:    
  Name: jenkins-slave  
  Namespace: jenkins  
  Labels: jenkins-slave  
  Usage: USe this node as much as possible  
Containers Template:   
  Name: jenkins-slave  
  Docker image: jenkinsci/jnlp-slave  
  Working directory: /home/jenkins  
