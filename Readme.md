# ARGO CD presentation

1. Installation of Minicube
Apply this command
```
https://minikube.sigs.k8s.io/docs/start/
```
Please install minicube on your computer at first

2. Install ArgoCD
- This will create a new namespace, argocd, where Argo CD services and application resources will live.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
- Download the latest Argo CD version from https://github.com/argoproj/argo-cd/releases/latest. More detailed installation instructions can be found via the CLI installation documentation.

Also available in Mac, Linux and WSL Homebrew:

```bash
brew install argocd
```

- Port Forwarding
Kubectl port-forwarding can also be used to connect to the API server without exposing the service.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
The API server can then be accessed using https://localhost:8080
```

- Login Using The CLI¶
The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using kubectl:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

- go to the console and write http://localhost:8080. you'll see the dashboard
- Username is `admin`. to be able to reach password apply this command
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
- Copy the password and past UI. Then you can see the dashboard

## Create you argocd application and deployment

### Create Deployment for nginx

- First we create namespace
```bash
kubectl create ns nginx
```
- Then we create deployment and service within `deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.io/serkangumus06/nginx:ryu
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx-myservice
  namespace: nginx
spec:
  ports:
  - name: nginx
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  sessionAffinity: None
  type: ClusterIP
```
- create this deployment and service
```bash
kubectl apply -f deploy.yaml
```

- On our local, we need to do port forwarding 
```bash
kubectl port-forward svc/nginx-myservice -n nginx 8082:80 &
```

- Show the web page http://localhost:8082

- Now we need to create argocd settings

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook-nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/serdarcw/argocdproject.git
    targetRevision: HEAD
    path: nginx-files
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

- create argocd app
```bash
kubectl apply -f application.yaml
```
- Go to the UI and control If app comes to the dashboard or not. and wait healthy status

- Then we can show the healt status, cdeployment componets, their sub components on UI

- Change the resplica number on Github and refresh on argocd UI

### Create Deployment for phonebook app

- First we create namespace
```bash
kubectl create ns phonebook
```
- Then we create deployment and service within ./kubernetes-files

- create this deployment and service
```bash
kubectl apply -f ./kubernetes-files
```

- Now we need to create argocd settings

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/serdarcw/argocdproject.git
    targetRevision: HEAD
    path: kubernetes-files
  destination:
    server: https://kubernetes.default.svc
    namespace: phonebook
  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```

- create argocd app
```bash
kubectl apply -f application.yaml
```
- Go to the UI and control If app comes to the dashboard or not. and wait healthy status. but two result and web_server has problem. Show their logs 

- Go to servers_configmap.yaml file and change ` mysql-service.default.svc.cluster.local` as ` mysql-service.phonebook.svc.cluster.local`

- Then we can show the healt status, cdeployment componets, their sub components on UI



Referances:
- https://redhat-scholars.github.io/argocd-tutorial/argocd-tutorial/02-getting_started.html

- https://minikube.sigs.k8s.io/docs/start/
- https://argo-cd.readthedocs.io/en/stable/getting_started/
- https://www.youtube.com/watch?v=MeU5_k9ssrs

