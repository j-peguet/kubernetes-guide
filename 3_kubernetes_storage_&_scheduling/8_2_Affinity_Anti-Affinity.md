# Affinity & Anti-Affinity

The difference between nodeAffinity and nodeSelectors is nodeAffinity allows you to use matchExpressions (for more complex rules).

* nodeAffinity - uses Labels on Nodes to make a scheduling decision with matchExpressions
  * requiredDuringSchedulingIgnoredDuringExecution - a pod is not scheduled if the defined rules are not evaluate to true
  * preferredDuringSchedulingIgnoredDuringExecution - if the rules are not evaluated to true, the pod is scheduled
* podAffinity - use labels and selectors to schedule Pods onto the same Node, Zone as some other Pod (with matchLabels or matchExpressions)
* podAntiAffinity - schedule Pods onto the
different Node, Zone as some other Pod

More infi here https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity

## Using Affinity to Control Pod Placement
In this example, I want to deploy a cache server, on a node that already have a web server (for lower latency).

```yaml
spec:
    containers:
    - name: hello-world-cache
    ...
    affinity:
        podAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - hello-world-web
                topologyKey: "kubernetes.io/hostname"
```

## Demo 1
Using Affinity to schedule Pods to Nodes

Create a file named deployment-affinity.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-web
  template:
    metadata:
      labels:
        app: hello-world-web
    spec:
      containers:
      - name: hello-world-web
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-cache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-cache
  template:
    metadata:
      labels:
        app: hello-world-cache
    spec:
      containers:
      - name: hello-world-cache
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - hello-world-web
            topologyKey: "kubernetes.io/hostname"
```

```bash
#Let's start off with a deployment of web and cache pods
#Affinity: we want to have always have a cache pod co-located on a Node where we a Web Pod
kubectl apply -f deployment-affinity.yaml

#Let's check out the labels on the nodes, look for kubernetes.io/hostname which
#we're using for our topologykey
kubectl describe nodes c1-node1 | head
kubectl get nodes --show-labels

#We can see that web and cache are both on the name node
kubectl get pods -o wide 

#If we scale the web deployment
#We'll still get spread across nodes in the ReplicaSet, so we don't need to enforce that with affinity
kubectl scale deployment hello-world-web --replicas=2
kubectl get pods -o wide 

#Then when we scale the cache deployment, it will get scheduled to the same node as the other web server
kubectl scale deployment hello-world-cache --replicas=2
kubectl get pods -o wide 

#Clean up the resources from these deployments
kubectl delete -f deployment-affinity.yaml
```

## Demo 2

Using Anti-Affinity to schedule Pods to Nodes

Let's create 2 files, deployment-antiaffinity.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-web
  template:
    metadata:
      labels:
        app: hello-world-web
    spec:
      containers:
      - name: hello-world-web
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - hello-world-web
            topologyKey: "kubernetes.io/hostname"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-cache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-cache
  template:
    metadata:
      labels:
        app: hello-world-cache
    spec:
      containers:
      - name: hello-world-cache
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - hello-world-web
            topologyKey: "kubernetes.io/hostname"
```

and deployment-antiaffinity-corrected.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-web
  template:
    metadata:
      labels:
        app: hello-world-web
    spec:
      containers:
      - name: hello-world-web
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - hello-world-web
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-cache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-cache
  template:
    metadata:
      labels:
        app: hello-world-cache
    spec:
      containers:
      - name: hello-world-cache
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - hello-world-web
            topologyKey: "kubernetes.io/hostname"
```

```bash
#Now, let's test out anti-affinity, deploy web and cache again. 
#But this time we're going to make sure that no more than 1 web pod is on each node with anti-affinity
kubectl apply -f deployment-antiaffinity.yaml
kubectl get pods -o wide

#Now let's scale the replicas in the web and cache deployments
kubectl scale deployment hello-world-web --replicas=4

#One Pod will go Pending because we can have only 1 Web Pod per node 
#when using requiredDuringSchedulingIgnoredDuringExecution in our antiaffinity rule
kubectl get pods -o wide --selector app=hello-world-web

#To 'fix' this we can change the scheduling rule to preferredDuringSchedulingIgnoredDuringExecution
#Also going to set the number of replicas to 4
kubectl apply -f deployment-antiaffinity-corrected.yaml
kubectl scale deployment hello-world-web --replicas=4

#Now we'll have 4 pods up an running, but doesn't the scheduler already ensure replicaset spread? Yes!
kubectl get pods -o wide --selector app=hello-world-web

#Let's clean up the resources from this demos
kubectl delete -f deployment-antiaffinity-corrected.yaml
```