# Annotations
Les annotations sont utilisé pour ajouter de petites information:
* Used to add additional information about your cluster resources
* Mostly used by people or tooling to make decisions
* Build, release, and image information exposed in easily accessible areas
* Saves you from having to write integrations to retrieve data from external data sources
* Non-hierarchical, key/value pair
* Can’t be used to query/select Pods or other resources
* Data is used for “other” purposes
* Keys can be up to 63 characters
* Values can be up to 256KB

### Imperative annotations
```bash
kubectl annotate pod nginx-pod owner=Anthony
kubectl annotate pod nginx-pod owner=NotAnthony --overwrite
```
### Declarative annotations
```yaml
apiVersion: v1
kind: Pod
    metadata:
        name: nginx-pod
        annotation: owner: Anthony
spec:
    containers:
        - name: nginx
        image: nginx
...
```
