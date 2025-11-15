## Setup and Deployment

The bootstrap.sh script contains all the commands to create the KinD cluster, taint nodes, and deploy the MySQL
StatefulSet and todoapp Deployment with the required affinity and anti-affinity rules.

Run the following commands to initialize the KinD cluster and deploy all resources:

```bash
kind create cluster --name todoapp-cluster --config cluster.yml

./bootstrap.sh
```

## Verify Nodes, Labels, and Taints

```bash
# List all nodes with labels
kubectl get nodes --show-labels

# Inspect taints on all nodes
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.taints}{"\n"}{end}'
```

Expected result:

Nodes labeled app=mysql should have a taint: app=mysql:NoSchedule.
Nodes labeled app=todoapp should exist for Deployment scheduling.
Control-plane node may have the default control-plane taint.

## Verify MySQL StatefulSet Scheduling

Check that MySQL pods are scheduled on nodes with the app=mysql label and respect pod anti-affinity

```bash
kubectl get pods -n mysql -o wide
kubectl describe pod <mysql-pod-name> -n mysql
```

Expected result:

MySQL pods are only scheduled on nodes labeled app=mysql.
No two MySQL pods are scheduled on the same node.
NodeAffinity ensures pods are scheduled on tainted nodes that tolerate app=mysql:NoSchedule.
PodAntiAffinity ensures distribution across nodes.

## Verify TodoApp Deployment Scheduling

Check that Deployment pods are scheduled on nodes with the app=todoapp label and respect pod anti-affinity:

```bash
kubectl get pods -n todoapp -o wide
kubectl describe pod <todoapp-pod-name> -n todoapp
```

Expected result:

Deployment pods are scheduled preferentially on nodes labeled app=todoapp.
PodAntiAffinity ensures multiple pods do not run on the same node.
If a node with the correct label is unavailable, pods may schedule on other nodes due to
preferredDuringSchedulingIgnoredDuringExecution.

## Validate Node Affinity and Pod Anti-Affinity

Check MySQL pod node assignment:

```bash
kubectl get pods -n mysql -o wide
kubectl describe pod <mysql-pod-name> -n mysql | grep -A5 "Affinity"
```

Check Deployment pod node assignment:

```bash
kubectl get pods -n todoapp -o wide
kubectl describe pod <todoapp-pod-name> -n todoapp | grep -A5 "Affinity"
```

Expected result:

Pods respect NodeAffinity rules.
Pods respect PodAntiAffinity rules.
MySQL pods are spread across nodes labeled app=mysql.
Deployment pods are spread across nodes labeled app=todoapp.

## 🧹 Cleanup

When finished, delete the entire KinD cluster to clean up all resources (pods, services, volumes, etc.).

```bash
kind delete cluster --name todoapp-cluster
```
