GitOps CI/CD with GitHub Actions and ArgoCD
Requirements
GitHub Actions
ArgoCD
Cloud Linux Instance (since GitHub Actions runs in the cloud) - AWS EC2 Ubuntu - t3.medium Running Minikube locally for creating a pipeline using GitHub Actions isn't feasible due to the nature of GitHub Actions itself. GitHub Actions runs in a cloud environment provided by GitHub, not on your local machine.
Docker
Kubernetes cluster (Minikube)
Kubectl
Create EC2 Instance & Set up Environment
Make the keypair executable:

chmod 600 keypair.pem
Connect with with putty or mobaxterm

Log in to Ubuntu EC2 Instance: Connect with with putty or mobaxterm for windows

or using mac terminal

ssh -i keypair.pem ubuntu@PublicIP 
Update & Upgrade System:

sudo apt update && sudo apt upgrade -y 
Install Docker

sudo apt  install docker.io -y 
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world 
docker version
systemctl status docker 
Install Kubectl

sudo snap install kubectl --classic 
kubectl version --client 
Install & Start Minikube

curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64 
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64 
minikube version
minikube start --driver=docker
kubectl get nodes 
Enable Ingress on Minikube

minikube addons enable ingress
Checkout Code into GitHub Actions

Click on the User Account
Click on Settings
Developer settings, and select Personal access tokens and Click Tokens (classic)
Generate new token, and select Generate new token (classic)
Note: actions-argocd-gitops00, Expiration: 30 days, and
Scopes (select the following): i. repo ii. workflow (For GitHub Actions) iii. admin:repo_hook (For Webhooks)
Generate token & save it somewhere safe Clone the GitHub Repository (if you want to work locally & push changes to remote repo)
git clone https://github.com/100foldtechnologies/ci_cd_pipeline.git
Create GitHub Actions Workflow Files & Directories

mkdir .github
cd .github
mkdir workflows
cd workflow
touch argocd-actions.yml
Create DockerHub Repository

Sign in to your DockerHub Account
Click "Create a repository"
Repository Name: gitHubActions-ArgoCD-00, Visibility: Public
Click Create.
Click on the User Account, Click on "Account settings", Click on "Personal access tokens"
Click "Generate new token", Expiration: 30 days, Access permissions: RWD. 7.Click Generate & save it somewhere safe
Create Environment Variables in GitHub

Click on the GitOps Repository and click on "Settings"
Click on "Secrets and variables" and select "Actions"
Under Repository secrets, click on "New repository secret"
 Name: DOCKERHUB_USERNAME
 Secret: 
 Add secret

 Name: DOCKERHUB_TOKEN
 Secret: 
 Add secret 
Set up ArgoCD Install ArgoCD in Minikube Namespace argocd

kubectl create namespace argocd 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml 
kubectl get pods -n argocd 
kubectl get svc -n argocd
Expose ArgoCD Server First, add port 8080 to Inbound Rules for the EC2 Instance

kubectl port-forward --address 0.0.0.0 svc/argocd-server 8080:443 -n argocd 
Get ArgoCD Initial Admin Password

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo 
To access ArgoCD on the browser:

PublicIP:8080
ARGOCD_USERNAME: admin
ARGOCD_PASSWORD: 
Install ArgoCD CLI (In GitHub Actions Pipeline)

curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/argocd
argocd version
Expose ArgoCD Service via NodePort (Ignore if using port forwarding as describe above)

kubectl get svc -n argocd
Replace Service Type ClusterIP with a NodePort (or LoadBalancer) - (default: 30000-32767)

kubectl edit svc argocd-server -n argocd
Add the NodePort (30007 and 30008 - Anywhere IPv4) to Inbound Rules for the EC2 Instance and

Rerun Port-forward command with the NodePort (30007) in a dedicated terminal

kubectl port-forward --address 0.0.0.0 svc/argocd-server 30007:80 -n argocd
OR Rerun Port-forward command with the NodePort (30008) in a dedicated terminal

kubectl port-forward --address 0.0.0.0 svc/argocd-server 30008:443 -n argocd
Login to ArgoCD (In GitHub Actions Pipeline)

Click Settings
Secrets and variables
Actions
New repository secret, to create new secrets for
  Name: ARGOCD_SERVER
    Value: PublicIP:30008
    Add secret 
 Name: ARGOCD_USERNAME
    Value: admin
    Add secret
 Name: ARGOCD_PASSWORD
    Value: 
    Add secret 
Configure Repository on ArgoCD UI (or CLI)

Go to Settings
Click on Repositories, and Connect Repo
Connection Method: Via HTTPS
Type: git
Project: default
Repository URL:
Username (optional):
Password (optional):
TLS Client Certificate (optional):
The remaining stuff optional. Leave as default and click CONNECT.
Create a New Application on ArgoCD UI (or with CLI below)

Click on Applications
Click New App
Application Name: argocd-github-actions
Project Name: default
Sync Policy: Automatic
Check Prune Resources & Self Heal
Repo URL: Click and select the Repo you attached earlier
Revision: main (this is the branch from which app is deployed)
Path: manifest
Cluster URL: Click and select the kubernetes.default.svc
Namespace:planning-app
(Alternatively) Using ArgoCD CLI to Create New Application

argocd app create my-app \
  --repo https://github.com/your-username/your-repo.git \
  --path manifest \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace planning-app 
Update Deployment file

Replace the Image with the Latest image built
ArgoCD UI will automatically Sync it & Deem it healthy
Inspect Deployment & Service on Terminal

kubectl get deploy -n argocd
kubectl get svc -n argocd
Automate ArgoCD Sync with GitHub Actions Workflow Pipeline

Add "argocd app sync argocd-github-actions" block to the pipeline
Commit changes and verify sync in the ArgoCD UI with Deploy, Svc, Pods, etc. 3.Inspect a Pod to see the port it listens. On EC2 Inbound rules, allow the port 3000 - AnywhereIPv4 - node app
Install NPM Modules on terminal & run the app
sudo apt install npm -y
sudo npm install -y 
On your browser:

PublicIP:3000
Run App Using Running Containers on ArgoCD kubectl get svc -n argocd kubectl port-forward --address 0.0.0.0 svc/myapp-service 8080:80 -n argocd On your browser:

PublicIP:8080/hello

Clean Up

kubectl delete ns argocd
minikube stop
minikube delete --all 
Terminate the EC2 Instance on AWS

About
No description, website, or topics provided.
Resources
 Readme
 Activity
 Custom properties
Stars
 0 stars
Watchers
 0 watching
Forks
 1 fork
Report repository
Releases
No releases published
Create a new release
Packages
No packages published
Publish your first package
Footer
