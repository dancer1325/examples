# Goal
* Create
  * Web frontend server,
  * an auto-provisioned persistent volume on GCE or Azure, and
  * an NFS-backed persistent claim
* Diagram
    * Web frontend Pod uses a PV based on NFS server, and
    * NFS server uses an auto provisioned PV from 
      * GCE PD or
      * AWS EBS or
      * Azure Disk

![alt text][nfs pv example]

[nfs pv example]: nfs-pv.png

# How to run?
* NFS server part
  * Create PVC
    * on GCE -> GCE PD PVC
      * `kubectl create -f examples/staging/volumes/nfs/provisioner/nfs-server-gce-pv.yaml`
    * on Azure -> Azure Disk PVC
      * `kubectl create -f examples/staging/volumes/nfs/provisioner/nfs-server-azure-pv.yaml`
  * `kubectl create -f examples/staging/volumes/nfs/nfs-server-deployment.yaml`
  * `kubectl exec -it podName -- sh`
    * Check it contains dummy `index.html`
  * `kubectl create -f examples/staging/volumes/nfs/nfs-server-service.yaml`
* Create NFS
  * `kubectl describe services nfs-server`
    * get the cluster IP of the server and
    * update 'nfs-pv.yaml'
      * We are hard coding it, right now
      * In next iterations, we can tie these together using the service names
  * `kubectl create -f examples/staging/volumes/nfs/nfs-pv.yaml` and `kubectl create -f examples/staging/volumes/nfs/nfs-pvc.yaml` 
* Setup the fake backend
  * `kubectl create -f examples/staging/volumes/nfs/nfs-busybox-deployment.yaml`
  * `kubectl exec podName -- cat /mnt/index.html`
    * Check that our mounts are working now
    * If you don't get anything 
      * Attempt: `kubectl describe services nfs-server` to check that it has endpoints listed (indicating the service was associated with a running pod)
* Setup the web server
  * `kubectl create -f examples/staging/volumes/nfs/nfs-web-deployment.yaml`
    * Pods serve the `index.html`
  * `kubectl create -f examples/staging/volumes/nfs/nfs-web-service.yaml`
    * Create a simple service to enable Pods to locate one another
  * `kubectl get services nfs-web`
    * Check the IP
  * `kubectl exec podName -- wget -qO- IP`
    * check that `nginx` is serving the data appropriately
    

# Web frontend component with the corresponding NFS based persistent volume
* [NFS Service and Deployment](nfs-web-deployment.yaml) 
* [NFS persistent volume](nfs-pv.yaml) and
* [NFS persistent volume claim](nfs-pvc.yaml)
