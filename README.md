# devops-cicd-production

**A Fully-Automated Production-Grade DevOps CI/CD Pipeline - Hands-On Project**

**Problem Statement:** How to deploy a Fully-Automated Production-Grade DevOps CI/CD Pipeline?

**Technology/Tools Stack that we will be using:**

1. We are going to use **GitLab** as our **SCM (Source Code Management)** and **CI (Continuous Integration)** Platform.

2. We will be using **ArgoCD** as our **CD (Continuous Deployment)** Platform.

3. We will deploy a **Frontend application** as Pods inside an **AWS EKS Cluster**.

4. The EKS Cluster will be deployed via **Terraform IaC**.
   
5. We will be utilizing **Snyk Tool** to scan our container images for security vulnerabilities.

6. The container images will reside on an **AWS ECR** private registry.

7. The **Devops CI/CD Deployment Pipeline** will be fully-automated.

**Application Architectural Diagram:**

![image](https://github.com/user-attachments/assets/8722ed58-a65c-4aee-b61c-ae2508f5a5a5)

**Step-By-Step Solution:**

**Step 1:** To administer the entire DevOps CICD pipeline, provision an AWS EC2 Instance (t2.micro) with Ubuntu 24.04 LTS AMI (or any VM on any cloud platform - it does not matter). This instance will function as a Bastion Host (Jump Server) to deploy resources. Please ensure that you attach an IAM role to the instance with following policies:

![image](https://github.com/user-attachments/assets/d5893ad3-62c5-4237-a7b8-d1987867b079)

Apart from the above AWS Managed policies, please attach a custom IAM policy (attached in the repo - name: eks-cluster-policy.json) to the IAM role. This is because we will interact with our AWS Account via CLI.

**Step 2:** SSH to this instance and follow the steps below to install Terraform and create AWS EKS cluster.

A. Follow the official documentation to install Terraform as the steps may change slightly in the future: https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli

B. Install git: _sudo apt install git -y_

C. _git clone https://github.com/bhavukm/eks-cluster.git_

D. _cd eks-cluster_

E. _terraform init_

F. _terraform apply --auto-approve_

G. Check your AWS account from UI and confirm the creation of the new EKS cluster

H. Install kubectl: 

_curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"_

_sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl_

J. Install awscli:

sudo apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

**Step 3:** Connect the created AWS EKS cluster with GitLab: https://docs.gitlab.com/user/clusters/agent/

A. Install the Kubernetes Agent for GitLab. Please ensure you are able to access the AWS EKS cluster: 

aws eks update-kubeconfig --region <aws-region> --name <cluster-name>

_kubectl get nodes_

B. Now, sign-up or use your email to login to https://gitlab.com

C. Create a project named "eks" on: https://gitlab.com

D. In the repository, in the default branch, create an agent configuration file at the root:

.gitlab/agents/awsekscluster/config.yaml

E. Leave the config.yaml file blank.

F. Register the agent with GitLab: Go inside the root directory of your "eks" project and click on the options as shown below:

![image](https://github.com/user-attachments/assets/3fd0da3c-714d-481f-92df-8d07a5188ad8)

![image](https://github.com/user-attachments/assets/d17b7484-cb78-442f-a37c-904cf745df13)

![image](https://github.com/user-attachments/assets/af8e10f0-3259-4db9-b800-6526462834d2)

Install helm:

_curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3_

_chmod 700 get_helm.sh_

_./get_helm.sh_

_helm repo add gitlab https://charts.gitlab.io_

_helm repo update_

_helm upgrade --install eks gitlab/gitlab-agent --namespace gitlab-agent-eks --create-namespace --set config.token=glagent-9zYw2RJK6qWdJRri5c3rFRGGD3sB-xwDPNMjydWNdx8eT8DVAA --set config.kasAddress=wss://kas.gitlab.com_

check the status on gitlab.com:

![image](https://github.com/user-attachments/assets/de111244-0b71-49cc-92fe-ba4f70ac315b)

**Step-4:** Now, install, configure and access argocd on the bastion host.

_kubectl create namespace argocd_

_kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.0.5/manifests/install.yaml_

_kubectl get all -n argocd_

_kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'_     #To expose the argocd server deployed inside the EKS cluster via a Load Balancer

Now, navigate to your AWS UI, and verify the creation of the Load Balancer for ArgoCD:

![image](https://github.com/user-attachments/assets/10ba06d5-6f12-46a0-99a4-34786d605212)

The LB DNS Name will be the AgoCD Server URL with default user set as admin

Retrieve the ArgoCD Server password: _kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d_

Now, go to your browser, and enter the crecdentials to access ArgoCD UI:

![image](https://github.com/user-attachments/assets/dfb572e7-35d4-4441-b572-513e03295049)

**Step-5:** Create an IAM user and assign Administrator access to it. Then, generate AWS Access keys and save it somewhere safely. These access keys will be used to authenticate to the GitLab server, where our 

application source code will be kept

![image](https://github.com/user-attachments/assets/6160dc97-0d32-405a-bb95-362a9a327ef4)

**Step-6:** Create an AWS ECR repository. In your AWS account, navigate to AWS ECR service and create a repo:

![image](https://github.com/user-attachments/assets/883a3cbd-9cfd-4ce9-8885-a70c2765903e)

This will be utilized to store our built container images

**Step-7:** Generate SNYK_TOKEN to utilize open-source Snyk container scanning tool to scan our images for security vulnerabilities

Browse to: https://app.snyk.io/login?redirectUri=L2FjY291bnQ_X2dsPTEqMWF2NXhvbipfZ2EqTXprM056QTBNamcxTGpFM05Ea3lPREEyTWprLipfZ2FfWDlTSDNLUDdCNCpjekUzTkRreU9EQTJNamtrYnpFa1p6RWtkREUzTkRreU9EQTJNekVrYWpVNEpHd3dKR2d3&from=snyk_auth_link

Or on your browser, type: "snyk token" and follow the steps to generate a snyk authentication token. This will be used to autheticate our GitLab server to Snyk server to scan our container images

![image](https://github.com/user-attachments/assets/4ffa5c87-4c4e-44b9-bcc1-e53142304bac)

Save the token.

**Step-8:** Now, follow the following steps to clone the application respository:

Fork the gitlab repository: git@gitlab.com:bhavukm/cicdnew.git
(https://gitlab.com/bhavukm/cicdnew.git)

![image](https://github.com/user-attachments/assets/c05198ec-e266-4fb0-9168-33229bdbb197)

mkdir -p cicdnew-2

cd cicdnew-2

git clone https://gitlab.com/bhavukm/cicdnew.git

cd cicdnew-2

cd argo-app

kubectl apply -f argo-app.yaml

Check on ArgoCD UI to see the created application:

![image](https://github.com/user-attachments/assets/cec3b4b5-bcbd-4c70-8cf3-64ace3972ea1)

ssh-keygen

![image](https://github.com/user-attachments/assets/1e943bd1-684e-42ea-b798-1b8c207cbbaf)

This ssh keypair is generated to authenticate your bastion host to the gitlab server.

cat ~/.ssh/id_ed25519.pub

Now, copy the ssh public key and go to your Gitlab server. I am using https://gitlab.com (create a new account using your gmail credentials)

click on edit profile:

![image](https://github.com/user-attachments/assets/84e089a1-420f-43c5-a5e9-3d683131bded)

Then, go to: SSH keys >> Add new key >> paste the ssh-public-key >> remove expiration date and Add Key:

![image](https://github.com/user-attachments/assets/3ce1c310-fe5e-4f3e-8da5-30cc9704f05d)

**step-9:** Since, we have a GitLab based CI/CD pipeleine that is automated, we will add all relevant environement variables to our GitLab project:

Go to project settings >> CI/CD

![image](https://github.com/user-attachments/assets/3cf0e0b9-f316-499d-9f4b-fc4bbb7adea3)

Click on variables, choose owner and save changes. This will allow our pipeline to have all the access it needs to use the variables in our pipeline stages

![image](https://github.com/user-attachments/assets/c3a2b8de-efaa-4dc6-90aa-1f711e3db824)

Now, add the all the following variables, one by one:

ARGOCD_PASSWORD = <ArgoCD Server's passoword for admin user>

ARGOCD_SERVER = <ArgoCD Server's Load Balancer DNS name>

ARGOCD_USERNAME = admin

AWS_ACCESS_KEY_ID = generated earlier for the IAM user to authenticate to the GitLab Server

AWS_DEFAULT_REGION = us-east-1

AWS_SECRET_ACCESS_KEY = generated earlier for the IAM user to authenticate to the GitLab Server

CLUSTER_NAME = demo-eks

DOCKER_HOST = tcp://docker:2375

ECR_REGISTRY = <AWS-Account-ID.dkr.ecr.us-east-1.amazonaws.com>

ECR_REPOSITORY = devopscicd

KUBERNETES_NAMESPACE = dev

SNYK_TOKEN = generated earlier to authenticate the GitLab server to Snyk server

![image](https://github.com/user-attachments/assets/6dea84e5-7562-473e-9db2-5e4cbb81dbd6)

cd ..

**step-10:** Test the DevOps CI/CD Automated Pipeline Deployment

git config --global --edit

![image](https://github.com/user-attachments/assets/39a9ee22-13d5-4da4-89dd-25ea448528b6)

save and quit

vim Dockerfile

Add a test line like this:

![image](https://github.com/user-attachments/assets/ba7fb284-7699-407b-bd6c-18d26a2388bd)

save and quit

git add Dockerfile

git commit -m "Test"

git push origin main

Note: If its asks for username and password, then follow the following steps:

git remote remove origin

git remote add origin git@gitlab.com:bhavukm/cicdnew-2.git

git push origin main

Check on https://gitlab.com if the pipeline ran successfully

![image](https://github.com/user-attachments/assets/eaf3950b-4d8c-409d-b335-e42e5c5dc766)

On the ArgoCD UI, the application should be updated:

![image](https://github.com/user-attachments/assets/5b1bcf63-9847-4bb3-8901-40e4768d82ea)





































