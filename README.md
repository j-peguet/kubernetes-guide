
# Guide Kubernetes 
<img alt="GitHub Version" src="https://img.shields.io/badge/Version-0.9-blue">
<img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/jaden37/kubernetes-guide?style=social">

<br>
<center>
<img src="kuber_logo.png" alt="drawing" width="200"/>
</center>

# 1. Installation et Configuration de Kubernetes

Installation et la configuration de VM pour l'utilisation de Kubernetes.

## 1. Installation des VM
 
* ###  1.1 [Installation des VM](1_install_&_configuration/1_install_VM.md)
* ###  1.2 [Configuration du Serveur Master](1_install_&_configuration/2_install_Master.md)
* ###  1.3 [Configuration d'une Node](1_install_&_configuration/3_install_Node.md)


## 2. Utilisation de Kubernetes

* ### 2.1 [Apprendre kubectl](1_install_&_configuration/4_use_Kubectl.md)
* ### 2.2 [Déploiement d'Application](1_install_&_configuration/5_application_deployment.md)
* ### 2.3 [Création de manifests](1_install_&_configuration/6_manifest_deployment.md)

# 2. Management de l'API Server et des Pods

Utilisation de l'API Server, le management d'objets avec différents systèmes de nommage 

## 1. Kubernetes API

* ###  1.1 [API Server](2_management_APIServer_&_Pod/1_API_Server.md)
* ###  1.2 [API Group](2_management_APIServer_&_Pod/2_API_Groups.md)
* ###  1.3 [API Version](2_management_APIServer_&_Pod/3_API_Version.md)
* ###  1.4 [API Request](2_management_APIServer_&_Pod/4_API_Request.md)

## 2. Nommages des objets Kubernetes

* ###  2.1 [Namespaces, Labels & Annotations](2_management_APIServer_&_Pod/5_0_Organizing_Objects.md)
  * [Namespaces](2_management_APIServer_&_Pod/5_1_Namespaces.md)
  * [Labels](2_management_APIServer_&_Pod/5_2_Labels.md)
  * [Annotations](2_management_APIServer_&_Pod/5_3_Annotations.md)

## 3. Running & managing pods

* ###  3.1 [Pods](2_management_APIServer_&_Pod/6_0_Pods.md)
  * [Single Pods](2_management_APIServer_&_Pod/6_1_Single_Pods.md)
  * [Multi Pods](2_management_APIServer_&_Pod/6_2_Multi_Pods.md)


# 3. Kubernetes storage & scheduling

Comment gérer les systèmes de stockage et influencer le scheduling des Pods.

## 1. Configuration et management du stockage

  * ### 1.1 [Persistent Storage](3_kubernetes_storage_&_scheduling/1_Persistent_storage.md)
  * ### 1.2 [Define PV & PVC](3_kubernetes_storage_&_scheduling/2_Define_PV_&_PVC.md)
  * ### 1.3 [Persistent Storage](3_kubernetes_storage_&_scheduling/3_Configure_server_storage.md)
  * ### 1.4 [Storage Class](3_kubernetes_storage_&_scheduling/4_Storage_Class.md)


## 2. Variables d'environnements, Secrets, ConfigMaps

  * ### 2.1 [Pods avec variables d'environnement](3_kubernetes_storage_&_scheduling/5_0_Pods_with_env.md)
    * [Environment Variables](3_kubernetes_storage_&_scheduling/5_1_Env_variables.md)
    * [Secrets](3_kubernetes_storage_&_scheduling/5_2_Secrets.md)
    * [ConfigMaps](3_kubernetes_storage_&_scheduling/5_3_ConfigMaps.md)
  * ### 2.2 [Accès a un registry privé](3_kubernetes_storage_&_scheduling/6_Access_to_private_registry.md)

## 3. Configuration et management du Scheduler

  * ### 3.1 [Scheduling](3_kubernetes_storage_&_scheduling/7_Scheduling.md)
  * ### 3.2 [Control Scheduling](3_kubernetes_storage_&_scheduling/8_0_Control_scheduling.md)
    * [Node Selector](3_kubernetes_storage_&_scheduling/8_1_Node_Selector.md)
    * [Affinity/ Anti-Affinity](3_kubernetes_storage_&_scheduling/8_2_Affinity_Anti-Affinity.md)
    * [Taint and Tolerations](3_kubernetes_storage_&_scheduling/8_3_Taints_&_Tolerations.md)
    * [Node Cordoning](3_kubernetes_storage_&_scheduling/8_4_Node_Cordining.md)
    * [Manual Scheduling](3_kubernetes_storage_&_scheduling/8_5_Manually_Scheduling.md)

Images provenant des cours d'[Antony Nocentino](https://app.pluralsight.com/profile/author/anthony-nocentino)