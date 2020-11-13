# Control Scheduling

They are many tools to influence where pods are scheduled inside of our cluster.

* [Node Selector](8_1_Node_Selector.md)
* [Affinity/ Anti-Affinity](8_2_Affinity_Anti-Affinity.md)
* [Taint and Tolerations](8_3_Taints_&_Tolerations.md)
* [Node Cordoning](8_4_Node_Cordining.md)
* [Manual Scheduling](8_5_Manually_Scheduling.md)

## Configuring Multiple Schedulers

* Implement your own scheduler - build a scheduler that's custom build for our workload
* Run multiple schedulers concurrently
* Define in your Pod Spec which scheduler you want
* Deploy your scheduler as a system Pod in the cluster

To look more closely a that process: https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/