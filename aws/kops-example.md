# 1. AWS configure. OR use env variable 
AWS ENV varialbe takes precedency and overrides aws configure
```
aws configure 

AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]: 
```

OR
```
export AWS_ACCESS_KEY_ID=[...]
export AWS_SECRET_ACCESS_KEY=[...]
export AWS_DEFAULT_REGION=us-east-2
```

# 2. Create IAM user
```
aws iam create-user --user-name kops
```
# 3. Create IAM Group
```
aws iam create-group --group-name kops
```

# 4. Attach permissions to groups
```
aws iam attach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess \
    --group-name kops

aws iam attach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess \
    --group-name kops

aws iam attach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess \
    --group-name kops

aws iam attach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/IAMFullAccess \
    --group-name kops
```

# 5. Add user to group
```
aws iam add-user-to-group --user-name kops --group-name kops
```

# 6. Create access key for user
```
aws iam create-access-key --user-name kops > kops-creds
cat kops-creds
```

# 7. Set new access key to env variables
```
export AWS_ACCESS_KEY_ID=NEWUSERKEY

export AWS_SECRET_ACCESS_KEY=NEWUSERSECRET
```

# 8. Availability zones
```
// check availability zones
aws ec2 describe-availability-zones --region $AWS_DEFAULT_REGION

// there are 3 availability zones in this region
{
    "AvailabilityZones": [
        {
            "State": "available", 
            "RegionName": "us-east-2", 
            "Messages": [], 
            "ZoneID": "use2-az1",
            "ZoneName": "us-east-2a"
        }, 
        {
            "State": "available", 
            "RegionName": "us-east-2", 
            "Messages": [],
            "ZoneID": "use2-az2", 
            "ZoneName": "us-east-2b"
        }, 
        {
            "State": "available", 
            "RegionName": "us-east-2", 
            "Messages": [], 
            "ZoneID": "use2-az3",
            "ZoneName": "us-east-2c"
        }
    ]
}
```

save zones
```
export ZONES=us-east-2a,us-east-2b,us-east-2c
```

# 9. Create SSH keys
// make a folder to store artifacts
```
mkdir -p cluster
cd cluster
```
create ssh private key via ec2 command
```
aws ec2 create-key-pair --key-name devops > devops.pem
// modify the file and only keep the KeyMaterial part containing RSA. remember to remove the \n and CR after ----
```

change permissions to the pem so only current user can read it, otherwise you can't generate the public key
```
// note in wsl, you have to copy this to /home/username before you can chmod 400 it. and generate the pub key there..
chmod 400 devops.pem
```

generate a public key
```
ssh-keygen -y -f devops.pem > devops.pub
```

# 10. Creating a Cluster: Creating S3 Bucket and Installing kops
// save name into an env variable
```
export NAME=devops.k8s.local
```
## 10.1 Create S3 bucket
```
// create a unique bucket name
export BUCKET_NAME=devops-$(date +%s)
aws s3api create-bucket --bucket $BUCKET_NAME --create-bucket-configuration LocationConstraint=$AWS_DEFAULT_REGION
export KOPS_STATE_STORE=s3://$BUCKET_NAME
```

# 11. Install kops
```
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x ./kops

sudo mv ./kops /usr/local/bin/
```

# 12. Choices
## 12.1 Making Choice with Master Nodes
Odd number > 1 for quorum. Min 3
## 12.2 Making Choice with Data Centers
3 datacenters close by for low latency and physical separation
## 12.3 Making Choice with Networking
* kubenet - kops’ default networking solution
  * routes for each node are configured in AWS VPC routing tables. Since those tables cannot have more than fifty entries, kubenet can be used in clusters with up to fifty nodes. If you’re planning to have a cluster bigger than that, you’ll have to switch to one of the previously mentioned CNIs.
* CNI - Container Network Interface (CNI) allows us to plug in a third-party networking driver.
* classic - deprecated in favor of kubenet
* external - custom, so discard this option
## 12.4 Making Choice with the Nodes’ Size
* t2.small is decent

# 13. Creating the cluster using kops
make sure kubectl is installed. do below to install kubectl if it's not installed. if it's installed, skip.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

create the cluster using kops
```
kops create cluster \
    --name $NAME \
    --master-count 3 \
    --node-count 1 \
    --node-size t2.small \
    --master-size t2.small \
    --zones $ZONES \
    --master-zones $ZONES \
    --ssh-public-key devops.pub \
    --networking kubenet \
    --kubernetes-version v1.22.9 \
    --yes
```

show cluster detail, then validate its readiness; it takes a while.
```
kops get cluster

NAME             CLOUD ZONES
devops.k8s.local aws   us-east-2a,us-east-2b,us-east-2c

kops validate cluster

Using cluster from kubectl context: devops.k8s.local

Validating cluster devops.k8s.local

INSTANCE GROUPS
NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
master-us-east-2a       Master  t2.small        1       1       us-east-2a
master-us-east-2b       Master  t2.small        1       1       us-east-2b
master-us-east-2c       Master  t2.small        1       1       us-east-2c
nodes-us-east-2a        Node    t2.small        1       1       us-east-2a
nodes-us-east-2b        Node    t2.small        0       0       us-east-2b
nodes-us-east-2c        Node    t2.small        0       0       us-east-2c

NODE STATUS
NAME                                            ROLE    READY
ip-172-20-105-252.us-east-2.compute.internal    master  True
ip-172-20-33-220.us-east-2.compute.internal     master  True
ip-172-20-36-54.us-east-2.compute.internal      node    True
ip-172-20-64-244.us-east-2.compute.internal     master  True

Your cluster devops.k8s.local is ready
```

# 14. Explaining what's in the cluster
When kops created EC2 instances, it first executed *nodeup*, which installed some packages. It ensure *docker*, *kubelet*, *protokube* are up and running.
* docker - runs containers
* kubelet - Kubernetes’ node agent. It runs on every node of a cluster, and its primary purpose is to run Pods; it ensures that the containers described in PodSpecs are running as long as they are healthy. It primarily gets the information about the Pods it should run through Kubernetes’ API server. As an alternative, it can get the info through files, HTTP endpoints, and HTTP servers.
* protokube - Specific to kops. primary responsibilities are to discover master disks, to mount them, and to create manifests. Some of those manifests are used by Kubelet to create system-level Pods and to make sure that they are always running. Besides starting the containers defined through Pods in the manifests (created by Protokube), Kubelet also tries to contact the API server which, eventually, is also started by it. Once the connection is established, Kubelet registers the node where it is running.

Docker, kubelet, Protokube runs on all *master* and *worker* nodes.

view system level pods
```
kubectl --namespace kube-system get pods

coredns-56dd667f7c-4622w                                               1/1     Running   0             25m
coredns-56dd667f7c-kl2hj                                               1/1     Running   0             27m
coredns-autoscaler-5c7694cfcc-4l2dj                                    1/1     Running   0             27m
dns-controller-5954d849fc-tq5gz                                        1/1     Running   0             27m
ebs-csi-controller-cfb5b5688-fsrhh                                     6/6     Running   0             27m
ebs-csi-controller-cfb5b5688-zc2bs                                     6/6     Running   0             27m
ebs-csi-node-4czpz                                                     3/3     Running   0             25m
ebs-csi-node-8gxrr                                                     3/3     Running   0             27m
ebs-csi-node-mjhdw                                                     3/3     Running   0             27m
ebs-csi-node-rfp8s                                                     3/3     Running   0             27m
etcd-manager-events-ip-172-20-105-252.us-east-2.compute.internal       1/1     Running   0             26m
etcd-manager-events-ip-172-20-33-220.us-east-2.compute.internal        1/1     Running   0             26m
etcd-manager-events-ip-172-20-64-244.us-east-2.compute.internal        1/1     Running   0             25m
etcd-manager-main-ip-172-20-105-252.us-east-2.compute.internal         1/1     Running   0             26m
etcd-manager-main-ip-172-20-33-220.us-east-2.compute.internal          1/1     Running   0             26m
etcd-manager-main-ip-172-20-64-244.us-east-2.compute.internal          1/1     Running   0             26m
kops-controller-9g9cw                                                  1/1     Running   0             27m
kops-controller-9hnzq                                                  1/1     Running   0             27m
kops-controller-jxszb                                                  1/1     Running   0             27m
kube-apiserver-ip-172-20-105-252.us-east-2.compute.internal            2/2     Running   1 (28m ago)   26m
kube-apiserver-ip-172-20-33-220.us-east-2.compute.internal             2/2     Running   2 (28m ago)   26m
kube-apiserver-ip-172-20-64-244.us-east-2.compute.internal             2/2     Running   1 (27m ago)   26m
kube-controller-manager-ip-172-20-105-252.us-east-2.compute.internal   1/1     Running   2 (28m ago)   26m
kube-controller-manager-ip-172-20-33-220.us-east-2.compute.internal    1/1     Running   3 (28m ago)   26m
kube-controller-manager-ip-172-20-64-244.us-east-2.compute.internal    1/1     Running   4 (27m ago)   25m
kube-proxy-ip-172-20-105-252.us-east-2.compute.internal                1/1     Running   0             25m
kube-proxy-ip-172-20-33-220.us-east-2.compute.internal                 1/1     Running   0             26m
kube-proxy-ip-172-20-36-54.us-east-2.compute.internal                  1/1     Running   0             25m
kube-proxy-ip-172-20-64-244.us-east-2.compute.internal                 1/1     Running   0             25m
kube-scheduler-ip-172-20-105-252.us-east-2.compute.internal            1/1     Running   0             26m
kube-scheduler-ip-172-20-33-220.us-east-2.compute.internal             1/1     Running   0             26m
kube-scheduler-ip-172-20-64-244.us-east-2.compute.internal             1/1     Running   0             26m
```

## 14.1 Master components
in addition to protokube, kubelet, docker
### 14.1.1 kube-apiserver
Kubernetes API Server accepts requests to create, update, or remove Kubernetes resources. It listens on ports 8080 (only reachable from the same server) and 443 (external).

Kubelet also uses 443 port to reach the API server and register itself as a node.

purpose is to validate and configure an api object (can be pods, services, rs, etc...) All the components in the cluster interact with the API Server for the operations that require a cluster-wide shared state. (etcd)

### 14.1.2 kube-controller-manager
in charge of running controllers

also in charge of Node Controllers responsible for monitoring servers and responding when one becomes unavailable.

### 14.1.3 kube-scheduler
Kubernetes Scheduler watches the API Server for new Pods and assigns them to a node. From there on, those Pods are run by Kubelet on the allocated node.

### 14.1.4 etcd
Kops creates an EBS volume for each etcd instance. It serves as its storage.

The shared state of the cluster is stored in etcd (key-value store where all cluster data is kept. HA via consistent data replication). It is split into two Pods, where *etcd-server* holds the state of the cluster and *etcd-server-events* stores the events.

### 14.1.5 kube-proxy
Kubernetes Proxy reflects Services defined through the API Server. It is in charge of TCP and UDP forwarding. It runs on all nodes of the cluster (both masters and workers).

### 14.1.6 dns-controller

DNS Controller allows nodes and users to discover the API Server.

## 14.2 Node components
in addition to protokube, kubelet, docker

### 14.2.1 kube-proxy
Kubernetes Proxy reflects Services defined through the API Server. It is in charge of TCP and UDP forwarding. It runs on all nodes of the cluster (both masters and workers).

# 15. Updating the cluster
see sub commands of kops edit
```
kops edit --help
```

*kops edit* will change the state stored in s3 bucket.
## 15.1 Editing the cluster
```
kops edit cluster --name $NAME
```
## 15.2 Editing the instance group instead
```
(deprecated. kops no longer groups instances nodes, since it creates 1 from every az)
kops edit ig --name $NAME nodes
```

## 15.3 updating the cluster after edit
tell kops we want to update the cluster to comply with the new state in s3 bucket
```
// without --yes, it's preview mode
kops update cluster --name $NAME --yes
```

kops will set kubectl context to the cluster that was just updated every time update happens.

also, as long as applications are scaled, kops rolling-updates will not cause downtime

## 15.3.1 Breakdown of the update process
* Kops retrieved the desired state from the S3 bucket.
* Kops sent requests to AWS API to change the values of the workers ASG.
* AWS modified the values of the workers ASG by increasing them by 1.
* ASG created a new EC2 instance to comply with the new sizing.
* Protokube installed Kubelet and Docker and created the manifest file with the list of Pods.
* Kubelet read the manifest file and run the container that forms the kube-proxy Pod (the only Pod on the worker nodes).
* Kubelet sent a request to the kube-apiserver (through the dns-controller) to register the new node and join it to the cluster. The information about the new node is stored in etcd.
* This process is almost identical to the one used to create the nodes of the cluster.

## 15.3.2 Verifying the update process
```
kops validate cluster
kubectl get nodes
```

## 15.3.3 Updating entire cluster, such as updating k8s version

```
kops edit cluster $NAME
kops update cluster $NAME
// the above update will not run if it will result in recreating the entire cluster
// have to use below command, because we cannot take down entire cluster => outage, loss of quorum->bad state

// preview
kops rolling-update cluster $NAME
// confirm
kops rolling-update cluster $NAME --yes
```


## deleting cluster
```
// need to switch to an IAM with access
kops delete cluster --name $NAME --yes

// delete s3 bucket
aws s3api delete-bucket --bucket $BUCKET_NAME
```