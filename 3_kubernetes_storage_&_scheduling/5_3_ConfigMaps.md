# ConfigMaps

* Key value pairs exposed into a Pod used application configuration settings
* Defining application or environment specific settings - connection strings, URLs, host names, credentials
  * A common use case is configured environments
  * All the information abstracted outside of the container
  * We can inject that configuration to sets up application (dev, QA, prod)
* Decouple application and Pod configurations
* Maximizing our container image’s portability
* Can be exposed as Environment Variables or Files

ConfigMaps must exist __before__ pod creation, or the container won't start up

## Using ConfigMaps in Pods
* Environment variables
  * valueFrom and envFrom
* Volumes and Files
  * Volume mounted inside a container
  * Single file or directory
  * Many files or directories
  * Volume ConfigMaps can be updated

## Defining ConfigMaps
### Imperative
```bash
kubectl create configmap appconfigprod \
--from-literal=DATABASE_SERVERNAME=sql.example.local \
--from-literal=BACKEND_SERVERNAME=be.example.local

kubectl create configmap appconfigqa \
--from-file=appconfigqa
```

### Declarative
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: appconfigprod
data:
    BACKEND_SERVERNAME: be.example.local
    DATABASE_SERVERNAME: sql.example.local
```

### Using ConfigMaps in Environment Variable
```yaml
spec:
    containers:
    - name: hello-world
      ...
      env:
      - name: DATABASE_SERVERNAME
        valueFrom:
            configMapKeyRef:
                name: appconfigprod
                key: DATABASE_SERVERNAME
      - name: BACKEND_SERVERNAME
        valueFrom:
            configMapKeyRef:
                name: appconfigprod
                key: BACKEND_SERVERNAME
```
It's a little big longer... we have better
```yaml
containers:
- name: hello-world
    ...
    envFrom:
        - configMapRef:
        name: appconfigprod
```
In this case each key and value pair inside of that secret. An environment variable is created, without need to specify them.


### Using ConfigMaps as Files
```yaml
spec:
    volumes:
        - name: appconfig
        configMap:
            name: appconfigqa
    containers:
    - name: hello-world
      ...
      volumeMounts:
      - name: appconfig
        mountPath: "/etc/appconfig"
```

## Demo 1

Creating ConfigMaps

Create a file appconfigqa
```
DATABASE_SERVERNAME="sqlqa.example.local"
BACKEND_SERVERNAME="beqa.example.local"
```

```bash
#Create a PROD ConfigMap
kubectl create configmap appconfigprod \
    --from-literal=DATABASE_SERVERNAME=sql.example.local \
    --from-literal=BACKEND_SERVERNAME=be.example.local

#Create a QA ConfigMap
#We can source our ConfigMap from files or from directories
#If no key, then the base name of the file
#Otherwise we can specify a key name to allow for more complex app configs and access to specific configuration elements
cat appconfigqa
kubectl create configmap appconfigqa \
    --from-file=appconfigqa

#Each creation method yeilded a different structure in the ConfigMap
kubectl get configmap appconfigprod -o yaml
kubectl get configmap appconfigqa -o yaml
```

## Demo 2
Using ConfigMaps in Pod Configurations
```bash
#First as environment variables
kubectl apply -f deployment-configmaps-env-prod.yaml

#Let's see or configured enviroment variables
PODNAME=$(kubectl get pods | grep hello-world-configmaps-env-prod | awk '{print $1}' | head -n 1)
echo $PODNAME

kubectl exec -it $PODNAME -- /bin/sh 
printenv | sort
exit
```

```bash
#Second as files
kubectl apply -f deployment-configmaps-files-qa.yaml

#Let's see our configmap exposed as a file using the key as the file name.
PODNAME=$(kubectl get pods | grep hello-world-configmaps-files-qa | awk '{print $1}' | head -n 1)
echo $PODNAME

kubectl exec -it $PODNAME -- /bin/sh 
ls /etc/appconfig
cat /etc/appconfig/appconfigqa
exit

#Our ConfigMap key, was the filename we read in, and the values are inside the file.
#This is how we can read in whole files at a time and present them to the file system with the same name in one ConfigMap
#So think about using this for daemon configs like nginx, redis...etc.
kubectl get configmap appconfigqa -o yaml

#Updating a configmap, change BACKEND_SERVERNAME to beqa1.example.local
kubectl edit configmap appconfigqa

kubectl exec -it $PODNAME -- /bin/sh 
watch cat /etc/appconfig/appconfigqa
exit

#Cleaning up our demp
kubectl delete deployment hello-world-configmaps-env-prod
kubectl delete deployment hello-world-configmaps-files-qa
kubectl delete configmap appconfigprod
kubectl delete configmap appconfigqa
```