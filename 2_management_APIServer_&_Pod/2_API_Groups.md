# API Groups
Les groupes sont des entités permettant de rassemblé des ressources.

Il y a deux grands groupes:
* Core API (Legacy Group) - ce sont les objets fondamental de kubernetes, quand la notion de groupe n'éxistait pas
* Named API Groups - ce sont tous les autres groupes crées par la suite (ex: storage)

## Core (Legacy)
C'est dans ce groupe que nous trouvons des objets fondamental comme:
* Pod
* Node
* Namespace
* PersistentVolume 
* PersistentVolumeClaim

## Named API Groups
Dans ces groupes nous trouvons des objets comme:
* apps - Deployment
* storage.k8s.io - StorageClass
* rbac.authorization.k8s.io - Role

Une liste plus détaillée est disponible ici: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#-strong-api-groups-strong-