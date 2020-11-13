# Accessing a Private Container Registry
* Secrets for application configuration
* Use Secrets to access a private container registry
* Want to access registries over the Internet
  * Docker Hub
  * Cloud based container registries
* Create a Secret of type docker-registry
* Enabling Kubernetes (kubelet) to pull the images from the private registry

## Demo
Pulling a Container from a Private Container Registry.

In this demo, don't forget to change the username here "nocentino"

First, you need a private repository in our registry, follow the directions here https://docs.docker.com/docker-hub/repos/#private-repositories

Create the file deployment-private-registry.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-private-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-private-registry
  template:
    metadata:
      labels:
        app: hello-world-private-registry
    spec:
      containers:
      - name: hello-world
        image: nocentino/hello-app:ps
        ports:
          - containerPort: 8080
      imagePullSecrets:
      - name: private-reg-cred
```


Pull a container, tag locally, and then push it into our private registry
```bash
#Then let's log into docker...I've already done this so it's using the configuration in config.json
sudo docker login 

#Let's pull down a hello-world image from gcr
sudo docker pull gcr.io/google-samples/hello-app:1.0

#Let's get our image ID so we can tag it locally
sudo docker image ls gcr.io/google-samples/hello-app

#tagging our image in the format your registry, image and tag
#You'll be using your own repository, so update that information here. 
sudo docker tag bc5c421ecd6c nocentino/hello-app:ps

#Now push that locally tagged image into our private registry at docker hub
#You'll be using your own repository, so update that information here. 
sudo docker push nocentino/hello-app:ps

#We need to adjust permissions on our config.json file, since I did a sudo docker login earlier...
sudo chown $(id -u):$(id -g) ~/.docker/
sudo chown $(id -u):$(id -g) ~/.docker/config.json
```

Create our secret that we'll use for our image pull...from our docker config.json
```bash
#This is of type generic, for docker-registry, look at line 48
kubectl create secret generic private-reg-cred \
    --from-file=.dockerconfigjson=/home/aen/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson

#...or if needed we can specify this explicitly using the following parameters
kubectl create secret docker-registry private-reg-cred \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=$USERNAME \
    --docker-password=$PASSWORD \
    --docker-email=$EMAIL
```

Ensure the image doesn't exist on any of our nodes...or else we can get a false positive
```bash
#you'll be using your own repository, so update that information here.
ssh jules@c1-node1 'sudo docker rmi nocentino/hello-app:ps'
ssh jules@c1-node2 'sudo docker rmi nocentino/hello-app:ps'

#Create a deployment using imagePullSecret in the Pod Spec.
kubectl apply -f deployment-private-registry.yaml

#Check out Containers and events section to ensure the container was actually pulled.
#This is why I made sure they were deleted from each Node above. 
kubectl describe pods hello-world

#clean up after our demo
kubectl delete -f deployment-private-registry.yaml
kubectl delete secret private-reg-cred
sudo docker rmi nocentino/hello-app:ps
```