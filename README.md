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

H. Install kubectl: curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

**Step 3:** Now, browse to https://gitlab.com/ and create a "New project".

**Step 4:** Connect the created AWS EKS cluster with GitLab: https://docs.gitlab.com/user/clusters/agent/

A. Install the Kubernetes Agent for GitLab: Please ensure you are able to access the AWS EKS cluster: kubectl get nodes



