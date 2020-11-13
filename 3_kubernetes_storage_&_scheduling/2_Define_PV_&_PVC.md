## Defining a Persistent Volume

* type - {nfs, fc, azureDisk, ...}
* capacity
* accessModes
* persistentVolumeReclaimPolicy - this parameter is Optional
* Labels

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-nfs-data
spec:
    capacity:
        storage: 10Gi
    accessModes:
        - ReadWriteMany
    nfs:
        server: 172.16.94.5
        path: "/export/volumes/pod"
```

## Defining a Persistent Volume Claim

* accessModes
* resources
* storageClassName
* selector

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-nfs-data
spec:
    accessModes:
        -ReadWriteMany
    resources:
        requests:
            storage: 10Gi
```

## Using Persistent Volume in Pods

```yaml
...
spec:
    volumes:
    - name: webcontent
      persistentVolumeClaim:
        claimName: pvc-nfs-data
    containers:
    - name: nginx
    ...
    volumeMounts:
    - name: webcontent
      mountPath: "/usr/share/nginx/html/web-app"
```