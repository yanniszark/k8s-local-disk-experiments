# Expose block volumes, make raid array in scylla pod

* Status: FAILED

## Steps:
1. Setup the devices you want to be discovered under a specified directory. 
eg. I want /dev/vdd to be used as a Block PersistentVolume. I create a symlink to /dev/vdd under a specified directory (like /mnt) which I then pass to the local volume provisioner parameters.

2. Setup the local volume provisioner for the cluster. If you don't have Helm installed, follow the [installation instructions](https://docs.helm.sh/using_helm/#installing-helm).
```bash
# Optional: Helm RBAC
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 

# Download the provisioner
git clone https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner.git

# Edit helm chart values:
# * Change volumeMode from Filesystem to Block
# * Change hostDir to the directory under which the local volumes are mounted (/mnt/disks on GKE)
# * Uncomment storageClass and reclaimPolicy in order for the StorageClass to be created automatically
vim sig-storage-local-static-provisioner/helm/provisioner/values.yaml

# Install the helm chart
helm install --name local-provisioner  sig-storage-local-static-provisioner/helm/provisioner
```

3. At this stage, we see the PVs being created. However, attempting to mount a Block PV inside of a process that is using it fails.