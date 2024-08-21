# Helm Workshop

## Helm structure

* Create helm directory.
* create helm chart.yaml.

```bash
mkdir /k8s/helm/templates/
touch /k8s/helm/Chart.yaml
```

* Chart.yaml

```yaml
apiVersion: v1
description: nginx Helm Chart 
name: nginx
version: 1.0.0
appVersion: 1.0.0
home: /
maintainers:
  - name: Developer
    email: dev@opsta.in.th
sources:
  - https://git.demo.opsta.co.th
```

* First helm install

```bash
# Check helm version
helm version
# Install local helm nginx 
helm install nginx /k8s/helm
# Show list helm
helm list
```

* Create helm values.yaml

```bash
touch k8s/helm/values.yaml
```

* helm value

```yaml
nginx:
  namespace: workshop-k8s-beer
  image: nginx
  tag: latest
  replicas: 1
  port: 80
service:
  targetPort: 80
  nodePort: 31000
  serviceType: NodePort
```

* Let's replace variable one-by-one with these object
  * {{ .Release.Name }}
  * {{ .Values.nginx.* }}

* Helm upgrade after edit parameter

```bash
helm upgrade nginx /k8s/helm -f /k8s/helm/values.yaml
```

* Change parameter of deployment template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Values.nginx.namespace }}
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.nginx.replicas }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: {{ .Release.Name }}
        image: {{ .Values.nginx.image }}:{{ .Values.nginx.tag }}
```

* Change parameter of service template

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Values.nginx.namespace }}
spec:
  type: {{ .Values.service.serviceType }}
  ports:
    - port: {{ .Values.nginx.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
      nodePort: {{ .Values.service.nodePort }}
  selector:
    app: {{ .Release.Name }}
```
* Edit replicas to 3 to values.yaml and upgrade helm

``` yaml
nginx:
  replicas: 3
```
* Upgrade helm to new version

```bash
helm upgrade nginx /k8s/helm -f /k8s/helm/values.yaml
```

```bash
# Check helm version
helm version
# Install local helm nginx 
helm install nginx /k8s/helm -f values.yaml
# Show list helm
helm list
# Show history 
helm history nginx
# Rollback to previous revision
helm rollback nginx 2
# See change
kubectl get pod
kubectl get deployment
# Clear resource
helm uninstall nginx
```

## Navigation

* previous: [kubernetes Workshop](kubernetes.md)