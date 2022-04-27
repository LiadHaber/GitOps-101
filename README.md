# K8s GitOps And Monitoring

## Prerequisites
 - Laptop/PC with 16GB+ of ram
 - K8S cluster, Im using docker-desktop single node cluster for this tutorial.(Kind is also a good and easy option - https://kind.sigs.k8s.io/docs/user/quick-start/)
 - Kubectl cli
 - Helm cli
## Features
 - Installing ArgoCD using helm
 - Adding our git repository to ArgoCD(We did say GitOps :) )
 - Installing the entrie prometheus stack using GitOps declartive methods and ArgoCD
 - Adding our own service-monitor
 - Adding ArgoCD dashboard to our grafana instance

## Hands on
Clone this repo to your local machine: 
```
git clone https://github.com/LiadHaber/GitOps-101.git && cd GitOps-101
```
Create argocd namesapce
```
kubectl apply -f argocd-ns.yaml
```
Add ArgoCD helm repo and install the chart
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --namespace argocd  argocd bitnami/argo-cd
```
Wait for ArgoCD server pod to be ready
```
kubectl wait pods -n argocd -l app.kubernetes.io/component=server --for condition=Ready
```
Get server username and password
```
echo "Username: \"admin\""
echo "Password: $(kubectl -n argocd get secret argocd-secret -o jsonpath="{.data.clearPassword}" | base64 -d)"
```
Port forward ArgoCD server
```
kubectl port-forward service/argocd-argo-cd-server 8080:80 -n argocd
```
Verify you can access ArgoCD by going to localhost:8080 and making sure you see the ArgoCD login page
![alt text ](https://redhat-scholars.github.io/argocd-tutorial/argocd-tutorial/_images/argocd-login.png)

Lets add our git repository
```
kubectl apply -f argocd-git-repo.yaml
````

You can verify the repository has be added by going to settings -> Repositories in the ArgoCD UI

Now this part is a little tricky, As of today(27/04/2022) the prometheus operator has an issue - One of the CRD's created is invalid(Annotaion too long).
You can try for yourself with - 
```
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml
```
1 workaround for this issue is installing the prometheus operator in 2 phases - 1 app will install the prometheus-operator CRD's and 1 app will install the prometheus resources

Create the prometheus app of apps
```
kubectl apply -f app-of-apps.yaml
```

This will create an app of apps consisting of alll resources managed by the prometheus operator - Such as grafana,prometheus, alertmanager etc.

For the purposes of this demo I disabled the node-exporter resource(Disables the node-exporter daemonset creation).
In addition - We want prometheus to be able to scrape ArgoCD metricsd so I added an additional service-monitor ArgoCD.
Both of this were supplied as values for the prometheus-stack app. 

Port-forward grafana service to access it
```
kubectl port-forward service/prometheus-stack-grafana -n monitoring 8081:80
```
You can go to Dashboards -> ArgoCD to view the dashboard. 

## Cleaning up 
If working with docker-desktop - Just reset the k8s cluster 
![alt text](https://birthday.play-with-docker.com/images/kubernetes-docker-desktop/settings-kubernetes.png)
If using kind
```
kind delete cluster --name <name of your cluster>
```