# Overview

Datastax Enteprise (DSE) is the Always-On, Distributed Cloud Database, Designed for Hybrid Cloud.

It has all the advantages of Apache Cassandra™ plus twice the performance, self-driving simplicity, analytics, search, graph, and advanced security.

This project will guide you though a deployment of DSE cluster on an EKS Kubernetes cluster and suggest some ways to deal with day 2 operations.

Provided here is a set of sample Kubernetes yamls to provision DataStax Enterprise in an EKS Kubernetes cluster.
It uses the "default" namespace in Kubernetes aa well as a sample EKS storage class definition. 
You should modify the yamls according to your own deployment requirements.

#### Prerequisites:
- Tools including eksctl and kubectl should already be installed on your machine.
- Kubernetes server's version is 1.8.x or higher. 
- an EKS cluster with 5 nodes and appropriate instances types
  - To create an EKS cluster see the instructions at https://github.com/scotthds/dse-eks/blob/master/eks/INSTALL.md

## Deploy DSE into EKS cluster

#### 1. Create required configmaps for DataStax Enterprise Statefulset and DataStax Enterprise OpsCenter Statefulset
```
$ git clone https://github.com/scotthds/dse-eks.git

$ cd kubernetes-dse

$ kubectl create configmap dse-config --from-file=common/dse/conf-dir/resources/cassandra/conf --from-file=common/dse/conf-dir/resources/dse/conf

$ kubectl create configmap opsc-config --from-file=common/opscenter/conf-dir/agent/conf --from-file=common/opscenter/conf-dir/conf --from-file=common/opscenter/conf-dir/conf/event-plugins

Sample opscenter.key and opscenter.pem are provided in the ssl folder for self-signed OPSC auth access.
$ kubectl create configmap opsc-ssl-config --from-file=common/opscenter/conf-dir/conf/ssl
```

#### 2. Create your own OpsCenter admin's password using K8 secret
You can update the [opsc-secrets.yaml file's admin_password's value](https://github.com/DSPN/kubernetes-dse/blob/dev-201808/common/secrets/opsc-secrets.yaml#L7) with your own base64 encoded password. Use this command **$ echo -n '\<your own password\>' | base64** to generate your base64 encoded password.
```
$ kubectl apply -f common/secrets/opsc-secrets.yaml 
```


#### 3. Deploy DSE + OpsCenter on EKS
```
$ kubectl apply -f eks/dse-suite.yaml
```

#### 4. Access the DataStax Enterprise OpsCenter managing the newly created DSE cluster

You can run the following command to monitor the status of your deployment.
```
$ kubectl get all
```
You can also run the following command to view if the status of **dse-cluster-init-job** has successfully completed.  It generally takes about 10 minutes to spin up a 3-node DSE cluster.
```
$ kubectl get job dse-cluster-init-job
```
Once complete, you can access the DataStax Enterprise OpsCenter web console to view the newly created DSE cluster by pointing your browser at https://<svc/opscenter-ext-lb's EXTERNAL-IP>:8443 with Username: admin and Password: **datastax1!** (if you use the default OpsCenter admin's password K8 secret)

#### 5. Tear down the DSE deployment
```
$ kubectl delete -f eks/dse-auite.yaml
$ kubectl delete pvc -l app=dse (to remove the dynamically provisioned persistent volumes for DSE)
$ kubectl delete pvc -l app=opscenter (to remove the dynamically provisioned persistent volumes for OpsCenter)
```

