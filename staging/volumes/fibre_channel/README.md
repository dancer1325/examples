# Consuming Fibre Channel Storage on Kubernetes

## Table of Contents
- [Definition](https://github.com/kubernetes/kubernetes/blob/master/api/openapi-spec/v3/apis__storage.k8s.io__v1_openapi.json#L242)
- [Example Parameters](#example-parameters)
- [Step-by-Step](#step-by-step)
- [Multipath Considerations](#multipath-considerations)

## Example Parameters

```yaml
 fc:
   # unique id in fbc devices
   targetWWNs:
     - '500a0982991b8dc5'
     - '500a0982891b8dc5'
   lun: 2
   # fstype   file system type
   fsType: ext4   # 4@ extended file system
   readOnly: true # write operations to the container mounting the volume
```

[API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#fcvolumesource-v1-core)

## Step-by-Step

### Set up a Fibre Channel Target

* Identify your Fibre Channel SAN Zone manager
  * Problems:
    * Problem1: TODO: How? 
* Allocate and mask LUNs
  * -> all hosts in the Kubernetes cluster can access them

### Prepare nodes in your Kubernetes cluster

* Install and configure a Fibre Channel initiator on the hosts within your Kubernetes cluster.
  * Problems:
      * Problem1: TODO: How?

### Create a Pod using Fibre Channel persistent storage

* `kubectl apply -f fc.yaml`
  * Problems:
    * Problem1: "MountVolume.WaitForAttach failed for volume "fc-vol" : no fc disk found"
      * Solution: TODO - Related to previous problems about how to set up all



If you connect to the console on the Kubernetes node that the pod has been assigned to you can see that the volume is mounted to the pod by running `mount | grep /var/lib/kubelet/plugins/kubernetes.io/fc/`

```console
# mount | grep /var/lib/kubelet/plugins/kubernetes.io/fc/
/dev/mapper/360a98000324669436c2b45666c567946 on /var/lib/kubelet/plugins/kubernetes.io/fc/500a0982991b8dc5-lun-2 type ext4 (relatime,seclabel,stripe=16,data=ordered)
  ```

## Multipath Considerations

To leverage multiple paths for block storage, it is important to perform
multipath configuration on the host.
If your distribution does not provide `/etc/multipath.conf`, then you can
either use the following minimalistic one:

```
defaults {
    find_multipaths yes
    user_friendly_names yes
}
```

or create a new one by running:

```console
$ mpathconf --enable
```

Finally you'll need to ensure to start or reload and enable multipath:

```console
$ systemctl enable --now multipathd.service
```

**Note:** Any change to `multipath.conf` or enabling multipath can lead to
inaccessible block devices as they will be claimed by multipath and
exposed as a device in /dev/mapper/*.


<!-- BEGIN MUNGE: GENERATED_ANALYTICS -->
[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/examples/volumes/fibre_channel/README.md?pixel)]()
<!-- END MUNGE: GENERATED_ANALYTICS -->
