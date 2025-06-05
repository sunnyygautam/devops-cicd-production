# devops-cicd-production
A production DevOps CICD Pipeline

**Problem Statement:** How to deploy a DevOps CICD production pipeline?

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

_sudo apt install unzip -y_

_curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install_

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
    

    









