# Storage Class

* Define tiers/classes of storage
* Enable Dynamic Provisioning
* Define infrastructure specific parameters
* Reclaim Policy

## Dynamic Provisioning Workflow

* Create a StorageClass - define the type of storage and reclaim policiy (optionnal)
* Create a PersistentVolumeClaim - pointing to the StorageClass
* Define Volume in Pod Spec - define a PersistentVolumeClaim pointing that PVC that we just created.
  * When the Pods start up, the PV is dynamically allocated

## Define StorageClass in Azure

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: managed-premium
parameters:
#Specific to our storage provisioner
    kind: Managed
    storageaccounttype: Premium_LRS
provisioner: kubernetes.io/azure-disk
```
Is this example no persistentVolumeClaimPolicy is defined. The default value is "delete", when I delete the PVC the underlying PV will be deleted.

## Dynamic Provisioning
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-azure-managed
spec:
    accessModes:
    - ReadWriteOnce
    storageClassName: managed-premium
    resources:
        requests:
            storage: 10Gi
```