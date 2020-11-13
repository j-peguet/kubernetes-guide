# Manually Scheduling a Pod

In some scenario you need to manually schedule a pod

* Scheduler populates nodeName
* If you specify nodeName in your Pod definition the Pod will be started on that node
* Node's name must exist
* Still subject to Node resource constraints

## Demo

Manually scheduling a Pod by specifying nodeName

Create pod.yaml file
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world-pod
spec:
  nodeName: 'c1-node1'
  containers:
  - name: hello-world
    image: gcr.io/google-samples/hello-app:1.0
    ports:
    - containerPort: 8080
```

```bash
kubectl apply -f pod.yaml

#Our Pod should be on c1-node1
kubectl get pod -o wide

#Let's delete our pod, since there's no controller it won't get recreated :(
kubectl delete pod hello-world-pod 

#Now let's cordon node1 again
kubectl cordon c1-node1

#And try to recreate our pod
kubectl apply -f pod.yaml

#You can still place a pod on the node since the Pod isn't getting 'scheduled', status is SchedulingDisabled
#Because we have specify the nodeName in the spec and bypass the scheduling process
kubectl get pod -o wide

#Can't remove the unmanaged Pod either since it's not managed by a Controller and won't get restarted in another node
kubectl drain c1-node1 --ignore-daemonsets 

#Let's clean up our demo, delete our pod and uncordon the node
kubectl delete pod hello-world-pod 
 
#Now let's uncordon node1 so it's able to have pods scheduled to it
kubectl uncordon c1-node1
```