# Setting Up a Kubeadm Cluster
Note: some of the names below will not work. You will have to provide unique names for GCP.
## Cloud setup
Create VPC network
```
$ gcloud compute networks create kubeadm-gcp --subnet-mode custom
```
Create subnet
```
$ gcloud compute networks subnets create kubernetes --network kubeadm-gcp --range 10.240.0.0/24
```
Create firewall rules
```
$ gcloud compute firewall-rules create kubeadm-nodeport \
   --allow tcp:30000–32767  \
   --network kubeadm-gcp \
   --source-ranges 0.0.0.0/0

$ gcloud compute firewall-rules create kubeadm-kube-api-server \
   --allow tcp:6443  \
   --network kubeadm-gcp \
   --source-ranges 10.240.0.0/24,10.200.0.0/16

$ gcloud compute firewall-rules create kubeadm-allow-ssh \
   --allow tcp:22  \
   --network kubeadm-gcp \
   --source-ranges 10.240.0.0/24,35.235.240.0/2
```
Create 2 instances (control plane and worker)
```
$ gcloud compute instances create kubeadm-controller\
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.10 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubeadm,controller

$ gcloud compute instances create kubeadm-worker \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.20 \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubeadm,worker
```
Disable swap ssh into both using `gcloud compute ssh <instance_name>`
```
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
exit
```
Then restart them.
## K8s bootstrapping
