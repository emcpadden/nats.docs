# Creating a Kubernetes Cluster

Below you will find examples of creating a small 3 node Kubernetes cluster to try NATS on multiple clouds.

## Google Kubernetes Engine

Use [gcloud](https://cloud.google.com/sdk/gcloud/) to create a 3 node [regional](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-regional-cluster) Kubernetes cluster on `us-west2`.

```bash
# Create a 3 node Kubernetes cluster. One node in each of the region's three zones.
gcloud container clusters create nats-k8s-cluster \
  --project $YOUR_GOOGLE_CLOUD_PROJECT \
  --region us-west2 \
  --num-nodes 1 \
  --machine-type n1-standard-2
```

Note that since this is a regional cluster we are specifying `--num-nodes 1` which will create a kubelet on 3 different zones. If you are creating a [single-zone cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster) but want 3 nodes then you have to specify `--num-nodes 3`.

## Amazon Kubernetes Service

The [eksctl](https://github.com/weaveworks/eksctl) is a very helpful tool to manage EKS clusters, you can find more docs on how to set it up [here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html).

```bash
# Create 3 node Kubernetes cluster
eksctl create cluster --name nats-k8s-cluster \
  --nodes 3 \
  --node-type=t3.large \
  --region=eu-west-1

# Get the credentials for your cluster
eksctl utils write-kubeconfig --name $YOUR_EKS_NAME --region eu-west-1
```

## Digital Ocean

You can use [doctl](https://github.com/digitalocean/doctl) to create a cluster as follows:

```bash
doctl kubernetes cluster create nats-k8s-nyc2 --count 3 --region nyc1
```

## Azure Kubernetes Service

Using [az](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) you can create a cluster like this:

```bash
# In case not done already, register to use some services:
az login
az provider register -n Microsoft.Network
az provider register -n Microsoft.Storage
az provider register -n Microsoft.Compute
az provider register -n Microsoft.ContainerService

# Create resource group and 3 node cluster
az group create --name nats --location westus
az aks create \
    --resource-group nats \
    --name nats  \
    --node-count 3 \
    --node-vm-size Standard_DS2_v2 \
    --load-balancer-sku basic \
    --enable-node-public-ip 

az aks get-credentials --resource-group nats --name nats
```

Eventually your cluster will be provided ExternalIPs that the NATS cluster can advertise to clients:

```text
kubectl get nodes -o wide
NAME                       STATUS   ROLES   AGE     VERSION    INTERNAL-IP   EXTERNAL-IP      OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-18657977-0   Ready    agent   5d16h   v1.13.12   10.240.0.6    52.191.186.114   Ubuntu 16.04.6 LTS   4.15.0-1060-azure   docker://3.0.7
aks-nodepool1-18657977-1   Ready    agent   5d17h   v1.13.12   10.240.0.4    52.229.11.82     Ubuntu 16.04.6 LTS   4.15.0-1060-azure   docker://3.0.7
aks-nodepool1-18657977-2   Ready    agent   5d17h   v1.13.12   10.240.0.5    13.77.149.235    Ubuntu 16.04.6 LTS   4.15.0-1060-azure   docker://3.0.7
```

