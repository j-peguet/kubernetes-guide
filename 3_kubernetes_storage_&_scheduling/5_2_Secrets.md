# Secrets
* Store sensitive information as Objects
* Retrieve for later use
* Passwords, API tokens, keys and certificates
* Safer and more flexible configurations (Pod Specs and Images)

## Properties of Secrets
* base64 encoded
* Encryption can be configured
* Stored in etcd
* Namespaced and can only be referenced by Pods in the same Namespace
* Unavailable Secrets will prevent a Pods from starting up

More info available here: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

They are 3 different types of secret in Kubernetes:
* Docker registry
* generic - create from local file/directory or value from command line
* TLS

## Create a Secret
```bash
kubectl create secret generic app1 \
    --from-literal=USERNAME=app1login \
    --from-literal=PASSWORD='S0methingS@Str0ng!'
```

you can also use --from-file or --from-directory

## Unsing secret in Pods
But how can our container based applications can be access to our secrets.
* Environment Variables
* Volumes or Files - tmpfs mounted file exposed
* Reference Secret __must__ be created and accessible for the Pod to start up - if is not available the Pod can't start

### Unsing secret in Environment Variables
```yaml
spec:
    containers:
    - name: hello-world
    ...
    env:
    - name: app1username
      valueFrom:
        secretKeyRef:
            name: app1
            key: USERNAME
    - name: app1password
      valueFrom:
        secretKeyRef:
            name: app1
            key: PASSWORD
```

It's a little big longer... we have better
```yaml
spec:
    containers:
    - name: hello-world
    ...
    envFrom:
    - secretRef:
        name: app1
```
In this case each key and value pair inside of that secret. An environment variable is created, without need to specify them.

### Unsing secret as Files
```yaml
spec:
    volumes:
    - name: appconfig
      secret:
        secretName: app1
    containers:
    ...
    volumeMounts:
        - name: appconfig
          mountPath: "/etc/appconfig"
```
In the directory container, for each key whey have a file named like the name of our pair. The value is in the file.

For ou example whe have to file:
* /etc/appconfig/USERNAME
* /etc/appconfig/PASSWORD

## Demo 1
Creating and accessing Secrets
```bash
#Generic - Create a secret from a local file, directory or literal value
#They keys and values are case sensitive
kubectl create secret generic app1 \
    --from-literal=USERNAME=app1login \
    --from-literal=PASSWORD='S0methingS@Str0ng!'


#Opaque means it's an arbitrary user defined key/value pair. Data 2 means two key/value pairs in the secret.
#Other types include service accounts and container registry authentication info
kubectl get secrets

#app1 said it had 2 Data elements, let's look
kubectl describe secret app1

#If we need to access those at the command line...
#These are wrapped in bash expansion to add a newline to output for readability
echo $(kubectl get secret app1 --template={{.data.USERNAME}} )
echo $(kubectl get secret app1 --template={{.data.USERNAME}} | base64 --decode )

echo $(kubectl get secret app1 --template={{.data.PASSWORD}} )
echo $(kubectl get secret app1 --template={{.data.PASSWORD}} | base64 --decode )
```

## Demo 2
Accessing Secrets inside a Pod

```bash
#As environment variables
kubectl apply -f deployment-secrets-env.yaml

PODNAME=$(kubectl get pods | grep hello-world-secrets-env | awk '{print $1}' | head -n 1)
echo $PODNAME

#Now let's get our enviroment variables from our container
#Our Enviroment variables from our Pod Spec are defined
#Notice the alpha information is there but not the beta information. Since beta wasn't defined when the Pod started.
kubectl exec -it $PODNAME -- /bin/sh
printenv | grep ^app1
exit
```
```bash
#Accessing Secrets as files
kubectl apply -f deployment-secrets-files.yaml

#Grab our pod name into a variable
PODNAME=$(kubectl get pods | grep hello-world-secrets-files | awk '{print $1}' | head -n 1)
echo $PODNAME

#Looking more closely at the Pod we see volumes, appconfig and in Mounts...
kubectl describe pod $PODNAME

#Let's access a shell on the Pod
kubectl exec -it $PODNAME -- /bin/sh

#Now we see the path we defined in the Volumes part of the Pod Spec
#A directory for each KEY and it's contents are the value
ls /etc/appconfig
cat /etc/appconfig/USERNAME
cat /etc/appconfig/PASSWORD
exit
```

```bash
#If you need to put only a subset of the keys in a secret check out this line here and look at items
#https://kubernetes.io/docs/concepts/storage/volumes#secret

#let's clean up after our demos...
kubectl delete secret app1
kubectl delete deployment hello-world-secrets-env
kubectl delete deployment hello-world-secrets-files
```