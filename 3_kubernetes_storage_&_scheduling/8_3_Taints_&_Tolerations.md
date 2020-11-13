# Taints and Tolerations

* Taints - ability to control which Pods are scheduled to Nodes
* Tolerations - allows a Pod to ignore a Taint and be scheduled as normal on Tainted Nodes
* Useful in scenarios where the cluster administrator needs to influence scheduling without depending on the user
  * key=value:effect
  * kubectl taint nodes c1-node1 key=MyTaint:NoSchedule
  * They are two different parameters
    * PreferNoSchedule
    * NoExecute

## Adding a Taint to a Nodes and a Toleration to a Pod

Assign a taint to a node

```bash
kubectl taint nodes c1-node1 key=MyTaint:NoSchedule
```

Add a toleration to the pod
```yaml
spec:
    containers:
    - name: hello-world
      image: gcr.io/google-samples/hello-app:1.0
      ports:
      - containerPort: 8080
    tolerations:
    - key: "key"
      operator: "Equal"
      value: "MyTaint"
      effect: "NoSchedule"
```

## Demo

Controlling Pods placement with Taints and Tolerations

Create two files: deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
```

and deployment-tolerations.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-tolerations
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-tolerations
  template:
    metadata:
      labels:
        app: hello-world-tolerations
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      tolerations:
      - key: "key"
        operator: "Equal"
        value: "MyTaint"
        effect: "NoSchedule"
```

```bash
#Let's add a Taint to c1-node1
kubectl taint nodes c1-node1 key=MyTaint:NoSchedule

#We can see the taint at the node level, look at the Taints section
kubectl describe node c1-node1

#Let's create a deployment with three replicas
kubectl apply -f deployment.yaml

#We can see Pods get placed on the non tainted nodes
kubectl get pods -o wide

#But we we add a deployment with a Toleration...
kubectl apply -f deployment-tolerations.yaml

#We can see Pods get placed on the non tainted nodes
kubectl get pods -o wide

#Remove our Taint
kubectl taint nodes c1-node1 key:NoSchedule-

#Clean up after our demo
kubectl delete -f deployment-tolerations.yaml
kubectl delete -f deployment.yaml
```