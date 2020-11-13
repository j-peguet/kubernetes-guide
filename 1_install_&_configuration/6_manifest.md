# Manifest

## Basic manifest - Pod

example

```yaml
# The version of the API Server (if a new one is created)
apiVersion: v1
# The object we want to define
kind: Pod
# Metadata to describe it
metadata:
    name: nginx-pod
spec:
    containers:
    - name: nginx
    image: nginx
```
Sauvegardons cette fonction dans un fichier nginx.yaml

Ensuite appliquer la commande suivante
```bash
kubectl apply -f nginx.yaml
```