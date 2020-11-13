# Node Cordoning

* Marks a Node as unschedulable
* Prevents new Pods from being scheduled to that Node
* Does not affect any existing Pods on the Node
* This is useful as a preparatory step before a Node reboot or maintenance
  * kubectl cordon c1-node2
* If you want to gracefully evict your Pods from  Node..., before your maintenance, you'll drain the node to ensure that workload gets scheduled somewhere else in the cluster
  * kubectl drain c1-node2 --ignore-daemonsets

## Demo

Node Cordoning

Check the file deployment.yaml in the demo of [Taints and Tolerations
](8_3_Taints_&_Tolerations.md)

```bash
#Let's create a deployment with three replicas
kubectl apply -f deployment.yaml

#Pods spread out evenly across the nodes
kubectl get pods -o wide

#Let's cordon c1-node2
kubectl cordon c1-node2

#That won't evict any pods...
kubectl get pods -o wide

#But if I scale the deployment
kubectl scale deployment hello-world --replicas=6

#c1-node2 won't get any new pods...one of the other Nodes will get an extra Pod here.
kubectl get pods -o wide

#Let's drain (remove) the Pods from c1-node2...
kubectl drain c1-node2 

#Let's try that again since daemonsets aren't scheduled we need to work around them.
kubectl drain c1-node2 --ignore-daemonsets

#Now all the workload is on c1-node1 and 2
kubectl get pods -o wide

#We can uncordon c1-node2, but nothing will get scheduled there until there's an event like a scaling operation or an eviction.
#Something that will cause pods to get created
kubectl uncordon c1-node2

#So let's scale that Deployment and see where they get scheduled...
kubectl scale deployment hello-world --replicas=9

#All three get scheduled to the cordoned node
kubectl get pods -o wide

#Clean up this demo...
kubectl delete deployment hello-world
```