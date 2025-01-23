#    Kubernetes Namespace, Services, and ReplicaSet Tutorial
## Namespace
A Namespace in Kubernetes serves as a resource isolation unit, helping to organize and manage resources within a cluster.

Namespace Isolation: By default, communication between different namespaces is restricted.

However, network policies can be implemented to enable communication between namespaces.

Useful Commands:
List all namespaces:

```bash
kubectl get ns
``` 
Create a new namespace:

```bash
kubectl create ns <namespace-name>
```
Delete an existing namespace:
```bash
kubectl delete ns <namespace-name>
```
Generate a YAML file for a pod within a specific namespace:

```bash
kubectl run <pod-name> -n <namespace-name> --image nginx --dry-run=client -o yaml
```
# Services
Services in Kubernetes work similarly to Docker port forwarding but with advanced features like kube-proxy and multiple service types (e.g., ClusterIP, NodePort, LoadBalancer). When you use the kubectl expose command, it creates a service to expose your pod.

Expose a Pod as a Service:
```bash
kubectl expose pod <pod-name> --port=8000 --target-port=80 --type=NodePort -n <namespace-name>
```
### Useful Commands:
List all services in a specific namespace:

```bash
kubectl get svc -n <namespace-name>
```
Describe a specific service:
```bash
kubectl describe svc <svc-name> -n <namespace-name>
```
Example output:

```text
Name:                     <service-name>
Namespace:                <namespace-name>
Labels:                   run=<service-name>
Annotations:              <none>
Selector:                 run=<service-name>
Type:                     NodePort
IP:                       10.109.240.18
Ports:                    8000/TCP
TargetPort:               80/TCP
NodePort:                 31933/TCP
Endpoints:                10.44.0.1:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
Delete a service:
```

```bash
kubectl delete svc <svc-name> -n <namespace-name>
```
Note: Deleting a pod does not automatically delete its associated service. If a pod is recreated with the same label, the existing service will continue to function.

## ReplicaSet
A ReplicaSet ensures that a specified number of identical pods are running at any time. If a pod fails, the ReplicaSet will automatically create a new one to maintain the desired number of replicas.

Sample ReplicaSet YAML:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

Apply the ReplicaSet YAML:

```bash
kubectl apply -f rs.yaml
```
Get a wide view of all pods:

```bash
kubectl get pods -o wide
```

Delete a specific pod:

```bash
kubectl delete pod <pod-name>
```
List all ReplicaSets:

```bash
kubectl get replicasets
```
Delete a ReplicaSet:

```bash
kubectl delete replicasets <replicaset-name>
```
Note: If any pod is deleted, Kubernetes will automatically create a new pod to maintain the desired replica count.

### Quick Reference for Common Commands
Here is a list of useful commands for managing namespaces, deployments, and services:

Create namespaces:

```bash
kubectl create ns alpha
kubectl create ns bravo
```
Create a deployment (example):

```bash
kubectl run alpha1 -n alpha --image=kiran2361993/kubegame:v1 --dry-run=client -o yaml
```
View available resources:

```bash
kubectl api-resources
```
Explain resources:

```bash
kubectl explain pod
kubectl explain <resource-type>
```

List all pods:

```bash
kubectl get pods
kubectl get pods -n alpha
```
Expose a pod as a service:

```bash
kubectl expose pod alpha1 --port=8000 --target-port=80 --type=NodePort
```
Describe a service:

```bash
kubectl describe svc alpha1
```