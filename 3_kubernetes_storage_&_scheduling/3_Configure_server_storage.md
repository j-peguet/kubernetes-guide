# Configure Storage Server

Install and configure nfs-kerner-server
```bash
#Install NFS Server and create the directory for our exports
sudo apt install -y nfs-kerner-server
sudo mkdir -p /export/volumes/pod

#Configure our NFS Export in /etc/export for /export/volumes. Using no_root_squash and no_subtree_check to 
#allow applications to mount subdirectories of the export directly.
sudo bash -c 'echo "/export/volumes  *(rw,no_root_squash,no_subtree_check)" > /etc/exports'
cat /etc/exports
sudo systemctl restart nfs-kernel-server.service
exit
```

On each Node (not the master server), install the NFS Client
```bash
sudo apt install -y nfs-common
```

Test the installation on one Node. If this test don't work create persistent volume in Kubernetes cannot be possible.
```bash
#Try to mount a volume
sudo mount -t nfs4 c1-storage1:/export/volumes /mnt/
mount | grep nfs
#Unmount it
sudo umount /mnt
exit
```

## Demo 1
Static Provisioning Persistent Volumes

create a file nfs.pv.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs-data
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: 192.168.4.103
    path: "/export/volumes/pod"
```

and a nfs.pvc.yaml file
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-data
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

```bash
#Create a PV with the read/write many and retain as the reclaim policy
kubectl apply -f nfs.pv.yaml


#Review the created resources, Status, Access Mode and Reclaim policy is set to Reclaim rather than Delete. 
kubectl get PersistentVolume pv-nfs-data

#Look more closely at the PV and it's configuration
kubectl describe PersistentVolume pv-nfs-data

#Create a PVC on that PV
kubectl apply -f nfs.pvc.yaml


#Check the status, now it's Bound due to the PVC on the PV. See the claim...
kubectl get PersistentVolume

#Check the status, Bound.
#We defined the PVC it statically provisioned the PV...but it's not mounted yet.
kubectl get PersistentVolumeClaim pvc-nfs-data
kubectl describe PersistentVolumeClaim pvc-nfs-data
```

Create some content in our storage server
```bash
ssh jules@c1-storage1
sudo bash -c 'echo "Hello from our NFS mount!!!" > /export/volumes/pod/demo.html'
cat /export/volumes/pod/demo.html
exit
```

Create a nfs.nginx.yaml file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-nfs-deployment
spec:  
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: webcontent
        persistentVolumeClaim:
          claimName: pvc-nfs-data
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: webcontent
          mountPath: "/usr/share/nginx/html/web-app"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-nfs-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
```

```bash
#Let's create a Pod (in a Deployment and add a Service) with a PVC on pvc-nfs-data
kubectl apply -f nfs.nginx.yaml
kubectl get service nginx-nfs-service
#Create an environment variable, that containt the IP address of the service
SERVICEIP=$(kubectl get service | grep nginx-nfs-service | awk '{ print $3 }')

#Check to see if our pods are Running before proceeding
kubectl get pods

#Let's access that application to see our application data...
curl http://$SERVICEIP/web-app/demo.html
```

Check the mouted volume in the container
```bash
#Check the Mounted By output for which Pod(s) are accessing this storage
kubectl describe PersistentVolumeClaim pvc-nfs-data
 
#If we go 'inside' the Pod/Container, let's look at where the PV is mounted
kubectl exec -it nginx-nfs-deployment-[tab][tab] -- /bin/bash
ls /usr/share/nginx/html/web-app
cat /usr/share/nginx/html/web-app/demo.html
exit
```

Check the mouted volume on the Node
```bash
#What node is this pod on?
kubectl get pods -o wide

#Let's log into that node and look at the mounted volumes...it's the kubelets job to make the device/mount available.
ssh c1-node[X]
mount | grep nfs
exit
```

Delete the pod, can we get our data ?
```bash
#Let's delete the pod and see if we still have access to our data in our PV...
kubectl get pods
kubectl delete pods nginx-nfs-deployment-[tab][tab]

#We get a new pod...but is our app data still there???
kubectl get pods


#Let's access that application to see our application data...yes!
curl http://$SERVICEIP/web-app/demo.html
```

## Demo 2
Control the Persistent Volume with Access Modes (Demo 1 is required).

```bash
#scale up the deployment to 4 replicas
kubectl scale deployment nginx-nfs-deployment --replicas=4

#Now let's look at who's attached to the pvc, all 4 Pods
#Our AccessMode for this PV and PVC is RWX ReadWriteMany
kubectl describe PersistentVolumeClaim 

#Now when we access our application we're getting load balanced across all the pods hitting the same PV data
curl http://$SERVICEIP/web-app/demo.html
```

```bash
#Let's delete our deployment
kubectl delete deployment nginx-nfs-deployment

#Check status, still bound on the PV...why is that...
kubectl get PersistentVolume 

#Because the PVC still exists...
kubectl get PersistentVolumeClaim

#Can re-use the same PVC and PV from a Pod definition...yes! Because I didn't delete the PVC.
kubectl apply -f nfs.nginx.yaml
```

```bash
#Our app is up and running
kubectl get pods 

#But if I delete the deployment
kubectl delete deployment nginx-nfs-deployment

#AND I delete the PersistentVolumeClaim
kubectl delete PersistentVolumeClaim pvc-nfs-data

#My status is now Released...which means no one can claim this PV
kubectl get PersistentVolume
```

```bash
#But let's try to use it and see what happend, recreate the PVC for this PV
kubectl apply -f nfs.pvc.yaml

#Then try to use the PVC/PV in a Pod definition
kubectl apply -f nfs.nginx.yaml

#My pod creation is Pending
kubectl get pods

#As is my PVC Status...Pending...because that PV is released and our Reclaim Policy is Retain
kubectl get PersistentVolumeClaim
kubectl get PersistentVolume

#Need to delete the PV if we want to 'reuse' that exact PV...to 're-create' the PV
kubectl delete deployment nginx-nfs-deployment
kubectl delete pvc pvc-nfs-data
kubectl delete pv pv-nfs-data
```
Now if I come back and recreate the PV, then I can get access to the underlying storage again with this deployment and its PVC and this process of deleting that PV puts that decision of do I want to be able to reuse that storage again into the administrator's hands __Kubernetes doesnt automatically make this decision for you.__

```bash
#If we recreate the PV, PVC, and the pods. we'll be able to re-deploy. 
#The clean up of the data is defined by the reclaim policy. (Delete will clean up for you, useful in dynamic provisioning scenarios)
#But in this case, since it's NFS, we have to clean it up and remove the files
#Nothing will prevent a user from getting this acess to this data, so it's imperitive to clean up. 
kubectl apply -f nfs.pv.yaml
kubectl apply -f nfs.pvc.yaml
kubectl apply -f nfs.nginx.yaml
kubectl get pods 


#Time to clean up for the next demo
kubectl delete -f nfs.nginx.yaml
kubectl delete pvc pvc-nfs-data
kubectl delete pv pv-nfs-data
```