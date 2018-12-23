# Using RAID0 on Kubernetes

## Environment: GKE

All the scripts and manifests are included in the manifests folder. All commands are assumed to run from inside the manifests folder.
```bash
cd manifests
```

1. Create a new nodepool using local-ssd with the number of disks that you want:
```bash
# Fill in gcloud command
```

2. Edit the daemonset and specify the number of disks to combine (NUM_DISKS). It will do the following:
    * Unmount every disk following the pattern /dev/ssd{i}
    * Use mdadm to combine them in a RAID0 array
    * Format the RAID device as xfs
    * Mount the RAID device under the specified RAID_DIR
```bash
# Change the value of NUM_DISKS to the number of disks you created
# in the gcloud command
vim raid-daemonset.yaml

# Deploy daemonset
kubectl apply -f raid-daemonset.yaml
```

3. Deploy the local-provisioner which will pick up the raid devices and present them as PersistentVolumes.
```bash
# Optional: Helm RBAC
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 

# Optional: delete previous installations of local-provisioner
helm ls -a
helm delete --purge <release_name>

# Install the local-provisioner
helm install --name local-provisioner provisioner
```

4. The raid devices should now get picked up by the local-provisioner. Check that the PVs are created:
```bash
kubectl get pv --watch
```
After the PVs are created, create a new Scylla cluster using the usual manifests. Make sure to:
 * Change the value of the disk capacity to match the PVs.
 * Change the `storageClassName` to `local-raid-disks`