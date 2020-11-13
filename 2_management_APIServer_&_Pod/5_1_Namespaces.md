# Namespaces
Le Namespaces permet:

* Ability to subdivide a cluster and its resources
* Conceptually a “virtual cluster”
* Deploy objects into a Namespace
* Resource isolation/organization (multitenants clusters)
* Security boundary for Role-based Access Controls
* Naming boundary (same name in different namespaces)
* A resource can be in only one namespace
* Has nothing to do with the concept of a Linux OS namespace

Permet de limiter les ressources en fonction d'un namespace (CPU, RAM, Disk). Il permet également de séparer des projets, applications, utilisateurs.

Ou de délimiter la prod, les test, la qualification.

## Working with namespace
* Create/Query/Delete
* Operate on objects in a Namespace
* Some objects are Namespaced... some aren’t
  * Resources are Namespaced...Pods, Controllers, Services
  * Physical things are not... PersistentVolumes, Nodes

Des namespaces existent déjà lors de l'installation du cluster
* default - namespace par défaut des objets, si aucun n'est spécifié
* kube-public - "readable" par tous les utilisateurs du cluster, utilisé pour stocker et partager des objets entre namespaces (ex: configmap)
* kube-system - objets système (pods, API Server, etcd, controller manager, kube-proxy)
* User Defined - permet à un utilisateurs de déployer son travail
* Two methods of declaration:
  * Imperatively with kubectl
    ```bash
    #Create Namespace
    kubectl create namespace playground1
    #Create Namespace
    kubectl run nginx --image=nginx --namespace playground1
    ```
  * Declaratively in a Manifest in YAML/JSON
    ```yaml
    #Create Namespace
        apiVersion: v1
        kind: Namespace
        metadata:
        name: playgroundinyaml
    #Create Object is this namespace
        apiVersion: apps/v1
        kind: Deployment
        metadata:
        namespace: playgroundinyaml
    ```

## Commands
```bash
#Get a list of all the namespaces in our cluster
kubectl get namespaces

#get a list of all the API resources and if they can be in a namespace
kubectl api-resources --namespaced=true | head
kubectl api-resources --namespaced=false | head

#Namespaces have state, Active and Terminating (when it's deleting)
kubectl describe namespaces

#Describe the details of an indivdual namespace
kubectl describe namespaces kube-system
```

```bash
#Get all the pods in our cluster across all namespaces. Right now, only system pods, no user workload.
#You can shorten --all-namespaces to -A
kubectl get pods --all-namespaces
kubectl get pods -A
#Get a list of the pods in the kube-system namespace
kubectl get pods --namespace kube-system
```

```bash
#Imperatively create a namespace
kubectl create namespace playground1
#Imperatively create a namespace...but there's some character restrictions. Lower case and only dashes. The regex is '[a-z0-9]([-a-z0-9]*[a-z0-9])?'
kubectl create namespace Playground1
```

```bash
#Declaratively create a namespace
cat namespace.yaml
kubectl apply -f namespace.yaml

#Get a list of all the current namespaces
kubectl get namespaces
```

```yaml
#Deployment.yaml file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  labels:
    app: hello-world
  namespace: playground1
spec:
  replicas: 4
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

```bash
#Start a deployment into our playground1 namespace
cat deployment.yaml
kubectl apply -f deployment.yaml

#Creating a resource imperitively
kubectl run hello-world-pod \
    --image=gcr.io/google-samples/hello-app:1.0 \
    --generator=run-pod/v1 \
    --namespace playground1

#Where are the pods?
kubectl get pods

#List all the pods on our namespace
kubectl get pods --namespace playground1
kubectl get pods -n playground1

#Get a list of all of the resources in our namespace
kubectl get all --namespace=playground1
```

```bash
#Try to delete all the pods in our namespace...this will delete the single pod.
#But the pods under the Deployment controller will be recreated.
kubectl delete pods --all --namespace playground1

#Get a list of all of the *new* pods in our namespace
kubectl get pods -n playground1

#Delete all of the resources in our namespace and the namespace and delete our other created namespace.
#This deletes the Deployment controller, the Pods...or really ALL resources in the namespaces
kubectl delete namespaces playground1
kubectl delete namespaces playgroundinyaml

#List all resources in all namespaces, now our Deployment is gone.
kubectl get all
kubectl get all --all-namespaces
```
