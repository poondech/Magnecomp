# Kubernetes Workshop

## Prerequisites

* WSL2 : https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview
* Helm : https://helm.sh/docs/intro/install/#from-chocolatey-windows
* kubectl : https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-nonstandard-package-tools
* VSCode : https://code.visualstudio.com/download

## Geting Started

* Get kubeconfig from Rancher

* Create directory .kube on current user
* Copy content kubeconfig to file .kube/config

```bash
# Check if it can connect to cluster
kubectl version
# View Cluster Information
kubectl cluster-info
# Show all pods
kubectl get pods --all-namespaces
```

## Setup your own namespace
* Namespace names must be unique

```bash
# Show all namespaces
kubectl get namespaces
# Show current cluster connection configuration
kubectl config get-contexts
# Create your own namespace
kubectl create namespace workshop-k8s-<xxx>
# Bypass Pod Security policy
kubectl label namespace <your-namespace> pod-security.kubernetes.io/enforce=privileged --overwrite
# Show your newly created  namespace
kubectl get namespace
# Set default namespace
kubectl config set-context $(kubectl config current-context) --namespace=<your-namespace>
kubectl config get-contexts
```

## Create Pod, Deployment and Service

```bash
# create  nginx deployment
kubectl create deployment nginx --image=nginx

# Set command history for record history
kubectl annotate deployment nginx kubernetes.io/change-cause='initial deployment'
# Show pod
kubectl get pod
# Show Deployment
kubectl get deployment
# Show detail of deployment
kubectl describe deployments nginx
# 
kubectl expose deployment nginx --type=NodePort --port=80 --target-port=80 
# Show Service
kubectl get service
```

## Scale service and change Docker image

```bash
# Scale pod to 3 replica
kubectl scale deployment nginx --replicas=3
# Show deployment and pods status
kubectl get deployment
kubectl get pod
# Change docker image to Apache and show pod status
kubectl set image deployment nginx nginx=httpd:2.4-alpine --record; kubectl get pod -w 
# See change
kubectl get deployment
kubectl describe deployment nginx
```

## Rollback Deployment

```bash
# Show history revision 
kubectl rollout history deployment nginx
# Rollback to previous revision
kubectl rollout undo deployment nginx
# See change
kubectl rollout history deployment nginx
kubectl describe deployment nginx
```

## Label and Selector

```bash
# Create new apache deployment
kubectl create deployment apache --image=httpd:2.4-alpine
# Set command history for record history
kubectl annotate deployment apache kubernetes.io/change-cause='inital apache deployment'
# Scale apache deployment to 3 replicas
kubectl scale deployment apache --replicas=3
# See change
kubectl get pod
kubectl get deployment
# See the label and selector
kubectl describe deployment nginx
kubectl describe service nginx
# See the label apache
kubectl describe deployment apache
# Set service nginx to select apache deployment label instead
kubectl set selector service nginx 'app=apache'
# Revert selector back
kubectl set selector service nginx 'app=nginx'
```

## Debug and troubleshooting

```bash
# Create pod for debug
kubectl create pod debug --image=ponetagon/network:debug
# Show pod
kubectl get pod
# Show log of pod
kubectl logs -f 
# Access to pod
kubectl exec -it debug -- sh
# Show service
kubectl get service
# Show status node
kubectl get nodes
# Show node information
kubectl describe node tlnw-prd-k8sw-1
```

## Manifest file

* Clear resources

```bash
# Show pod and deployment
kubectl get pod
kubectl get deployment
# Show service
kubectl get svc
# Delete pod and Deployment
kubectl delete deployment nginx apache
# Delete service
kubectl delete service nginx
# See change
kubectl get pod
kubectl get deployment
kubectl get svc
```

## Deploy pod manifest file

* Create manifest file 01-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: <your-namespace>
spec:
  containers:
  - name: busybox
    image: busybox
    command:
    - sleep
    - "3600"
```

* Create pod from manifest file

```bash
# Apply manifest file to cluster
kubectl apply -f 01-pod.yaml
# Access to pod
kubectl exec -it busybox -- sh
ping www.google.com
```

## Helper syntax manifest file

```bash
# Helper manifest file
kubectl api-resources
# Show manifest syntax for pod
kubectl explain pod
# Show manifest syntax for pod.spec
kubectl explain pod.spec
# Show manifest syntax for deployment
```

## Deployment and Service manifest file

* Create 02-apache-deployment.yaml file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: <your-namespace>
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```

* Create 02-nginx-service.yaml file

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: <your-namespace>
spec:
  type: NodePort
  ports:
  - port: 80
    protocal: TCP
    targetPort: 80
    nodePort: 31000
  selector:
    app: nginx
```

* Create deployment and service from manifest file

```bash
# Create nginx deloyment from manifest file
kubectl apply -f 02-nginx-deployment.yaml
# Create nginx service from manifest file
kubectl apply -f 02-nginx-service.yaml
# See resource
kubectl get pod
kubectl get deployment
kubectl get svc
# Clear resource
kubectl delete -f 01-pod.yaml -f 02-nginx-deployment.yaml -f 02-nginx-service.yaml
```

## Navigation

* Next: [Helm Workshop](helm.md)
