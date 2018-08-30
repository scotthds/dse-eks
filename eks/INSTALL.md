# Creating an EKS cluster

Please read the EKS getting started guide before creating the EKS cluster and deploying DSE in the steps [here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)

Amazon EKS is available in the following Regions at this time:
US West (Oregon) (us-west-2)
US East (N. Virginia) (us-east-1)

It is assumed that you have the latest version of the AWS CLI installed. If not, follow the instructions [here]( https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-bundle.html)

Running kubectl version --short --client should show a 1.10 or higher version
Client Version: v1.10.3

## EKSCTL
eksctl is a simple CLI tool for creating clusters on EKS - Amazon's new managed Kubernetes service for EC2. check out the project [here](https://github.com/weaveworks/eksctl)

*** The instructions below are for a basic 5 node kubernetes cluster where a new VPC is created for the cluster ***

Download the latest release and move into /usr/local/bin

curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

You will need to have AWS API credentials configured. What works for AWS CLI should be sufficient.
Create a cluster and write cluster credentials to a file other than the default.

eksctl create cluster --name=cluster-sky --nodes=5 --node-type=c5.xlarge --kubeconfig=./kubeconfig.cluster-sky.yaml

Output will look like below

2018-08-21T17:29:03-04:00 [ℹ]  setting availability zones to [us-west-2b us-west-2c us-west-2a]

2018-08-21T17:29:03-04:00 [ℹ]  creating EKS cluster "cluster-sky" in "us-west-2" region

2018-08-21T17:29:03-04:00 [ℹ]  creating ServiceRole stack "EKS-cluster-sky-ServiceRole"

2018-08-21T17:29:03-04:00 [ℹ]  creating VPC stack "EKS-cluster-sky-VPC"

2018-08-21T17:29:44-04:00 [✔]  created ServiceRole stack "EKS-cluster-sky-ServiceRole"

2018-08-21T17:30:24-04:00 [✔]  created VPC stack "EKS-cluster-sky-VPC"

2018-08-21T17:30:24-04:00 [ℹ]  creating control plane "cluster-sky"

2018-08-21T17:52:04-04:00 [✔]  created control plane "cluster-sky"

2018-08-21T17:52:04-04:00 [ℹ]  creating DefaultNodeGroup stack "EKS-cluster-sky-DefaultNodeGroup"

2018-08-21T17:55:45-04:00 [✔]  created DefaultNodeGroup stack "EKS-cluster-sky-DefaultNodeGroup"

2018-08-21T17:55:45-04:00 [✔]  all EKS cluster "cluster-sky" resources has been created

2018-08-21T17:55:46-04:00 [✔]  saved kubeconfig as "./kubeconfig.cluster-sky.yaml"
2018-08-21T17:55:46-04:00 [ℹ]  the cluster has 0 nodes

2018-08-21T17:55:46-04:00 [ℹ]  waiting for at least 5 nodes to become ready

2018-08-21T17:56:12-04:00 [ℹ]  the cluster has 5 nodes

2018-08-21T17:56:12-04:00 [ℹ]  node "ip-192-168-140-232.us-west-2.compute.internal" is ready

2018-08-21T17:56:12-04:00 [ℹ]  node "ip-192-168-174-216.us-west-2.compute.internal" is ready

2018-08-21T17:56:12-04:00 [ℹ]  node "ip-192-168-205-33.us-west-2.compute.internal" is ready

2018-08-21T17:56:12-04:00 [ℹ]  node "ip-192-168-78-243.us-west-2.compute.internal" is ready

2018-08-21T17:56:12-04:00 [ℹ]  node "ip-192-168-99-119.us-west-2.compute.internal" is ready

2018-08-21T17:56:12-04:00 [✖]  heptio-authenticator-aws not installed

2018-08-21T17:56:12-04:00 [ℹ]  cluster should be functional despite missing (or misconfigured) client binaries

2018-08-21T17:56:12-04:00 [✔]  EKS cluster "cluster-sky" in "us-west-2" region is ready



### Get cluster info using eksctl
eksctl get cluster --name=cluster-sky  --region=us-west-2

Output will look like the following

NAME		VERSION	STATUS	CREATED			VPC			SUBNETS						SECURITYGROUPS

cluster-sky	1.10	ACTIVE	2018-08-21T19:21:07Z	vpc-01748d5f53039cb5a	subnet-000d959968d33f35b,subnet-0103b602fbe63230d,subnet-0462dbc76660a4059	sg-02ec64fd163af0c7b


At this point you should be able see cluster information in the AWS console at https://us-west-2.console.aws.amazon.com/eks/home?region=us-west-2#/clusters




### Configure Kubernetes for Amazon EKS
Please read about configuring kubectl before executing the following steps [here](https://docs.aws.amazon.com/eks/latest/userguide/configure-kubectl.html)

### Install kubectl locally
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/darwin/amd64/kubectl (this example is for macOS clients)

cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH

### Check kubectl version
kubectl version --short --client

Install aws-iam-authenticator for Amazon EKS

curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/darwin/amd64/aws-iam-authenticator

chmod +x ./aws-iam-authenticator

cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=~/bin:$PATH


### Test aws-iam-authenticator
aws-iam-authenticator help

### Create a kubeconfig for Amazon EKS
Please read about creating a kubeconfig before executing the following steps. https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html

kubeconfig.cluster-sky.yaml created earlier can be used a starting point.
Create the default ~/.kube directory if it does not already exist.

mkdir -p ~/.kube

Replace "command: heptio-authenticator-aws" with  "command: aws-iam-authenticator" in kubeconfig.cluster-sky.yaml        
Copy kubeconfig.cluster-sky.yaml ~/.kube/config

### Test kubectl
kubectl get nodes

NAME                                            STATUS    ROLES     AGE       VERSION

ip-192-168-140-231.us-west-2.compute.internal   Ready     <none>    10m       v1.10.3
  
ip-192-168-176-84.us-west-2.compute.internal    Ready     <none>    10m       v1.10.3
  
ip-192-168-216-128.us-west-2.compute.internal   Ready     <none>    10m       v1.10.3
  
ip-192-168-223-223.us-west-2.compute.internal   Ready     <none>    10m       v1.10.3
  
ip-192-168-76-62.us-west-2.compute.internal     Ready     <none>    10m       v1.10.3
  

## Deploy DSE on EKS

Follow the steps [here](https://github.com/scotthds/dse-eks/blob/master/README.md) to deploy DSE on your EKS cluster


### Delete EKS cluster
eksctl delete cluster --name=cluster-sky  --region=us-west-2

Delete any cloud formation stacks remaining that are associated with this cluster



## Managing Users or IAM Roles for your Cluster
In order to add more users to the cluster follow the steps below. Please read https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html first.


### Edit aws-auth ConfigMap for your cluster
kubectl edit -n kube-system configmap/aws-auth

Add a section for new users similar to below
mapUsers: |
  - userarn: arn:aws:iam::819041172558:user/scott.hendrickson
    username: scott.hendrickson
    groups:
      - system:masters
