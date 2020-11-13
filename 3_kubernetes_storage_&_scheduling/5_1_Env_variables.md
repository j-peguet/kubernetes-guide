# Environment Variables
They are two types of environment variables
* User Defined
  * Pod Spec for each container
  * Defined inside the container image
  * Defined in name/value or valueFrom
* System Defined
  * Names of all Services available at the time the Pod was created
  * Defined at container startup
  * They can't be updated once the Pod is created

## Defining Environment Variables
```yaml
spec:
    containers:
    - name: hello-world
      image: gcr.io/google-samples/hello-app:1.0
      env:
      - name: DATABASE_SERVERNAME
        value: "sql.example.local"
      - name: BACKEND_SERVERNAME
        value: “be.example.local"
```

## Demo
Passing environments variables in our containers
Let's create 2 files deployment-alpha.yaml & deployment-beta.yaml (replace only alpha by beta)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-alpha
  template:
    metadata:
      labels:
        app: hello-world-alpha
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        env:
        - name: DATABASE_SERVERNAME
          value: "sql.example.local"
        - name: BACKEND_SERVERNAME
          value: "be.example.local"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world-alpha
spec:
  selector:
    app: hello-world-alpha
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

```bash
#Create two deployments, one for a database system and the other our application.
#I'm putting a little wait in there so the Pods are created one after the other.
kubectl apply -f deployment-alpha.yaml
sleep 5
kubectl apply -f deployment-beta.yaml

#Let's look at the services
kubectl get service
```

Create a local variables with the name of the Pod
```bash
#Now let's get the name of one of our pods
PODNAME=$(kubectl get pods | grep hello-world-alpha | awk '{print $1}' | head -n 1)
echo $PODNAME
```

```bash
#Inside the Pod, let's read the enviroment variables from our container
#Notice the alpha information is there but not the beta information. Since beta wasn't defined when the Pod started.
kubectl exec -it $PODNAME -- /bin/sh 
printenv | sort
exit

#If you delete the pod and it gets recreated, you will get the variables for the alpha and beta service information.
kubectl delete pod $PODNAME

#Get the new pod name and check the environment variables...the variables are define at Pod/Container startup.
PODNAME=$(kubectl get pods | grep hello-world-alpha | awk '{print $1}' | head -n 1)
kubectl exec -it $PODNAME -- /bin/sh -c "printenv | sort"
```

```bash
#If we delete our service and deployment 
kubectl delete deployment hello-world-beta
kubectl delete service hello-world-beta

#The enviroment variables stick around...to get a new set, the pod needs to be recreated.
kubectl exec -it $PODNAME -- /bin/sh -c "printenv | sort"

#Let's clean up after our demo
kubectl delete -f deployment-alpha.yaml
```