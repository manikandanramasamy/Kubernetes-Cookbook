# Kubernetes Deployment, DaemonSet, StatefulSet, and Persistent Volumes Guide
Deployment
To create a deployment in Kubernetes, use the following command:

```bash
kubectl create deployment <deployment-name> --image=nginx:1.26 --dry-run=client -o yaml
```
This will generate a deployment YAML file for the given deployment. A sample deployment definition might look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: <deployment-name>
  name: <deployment-name>
  namespace: <namespace-name>
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <deployment-name>
  strategy: {}
  template:
    metadata:
      labels:
        app: <deployment-name>
    spec:
      containers:
      - name: nginx
        image: nginx:1.26

```
### Important Notes:
kubectl run <pod-name> --image=nginx:1.26 will not create a ReplicaSet but just a pod or deployment.
If you want to add environment variables or annotations to your deployment, you can include them in the YAML like so:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: <deployment-name>
  name: <deployment-name>
  namespace: <namespace-name>
  annotations:
    version: "v1.2.3"
    description: "This is an example pod with annotations."
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <deployment-name>
  template:
    metadata:
      labels:
        app: <deployment-name>
    spec:
      containers:
        - name: nginx
          image: nginx:1.26
          env:
            - name: APP_ENV
              value: "production"
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: my-db-secret
                  key: host
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: my-db-secret
                  key: password
```
Setting Secrets for Environment Variables:
Before creating the deployment, you need to create the necessary secrets:

```bash
kubectl create secret generic my-db-secret \
  --from-literal=host=my-database-host \
  --from-literal=password=secretpass
```
After creating the deployment, you can view the environment variables with:

```bash
kubectl exec -it <deployment-name>-<pod-id> -- bash

```
### Exposing the Deployment as a Service:
After deployment, expose the service:

```bash
kubectl expose deployment <deployment-name> --name <service-name> --port 8081 
--target-port 80 --type NodePort
```

### Rolling Updates
For rolling updates, you can modify the deployment YAML file with the rollingUpdate strategy:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: <deployment-name>
  name: <deployment-name>
spec:
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 100%
      maxUnavailable: 33%  # Usually 33% or 1
  selector:
    matchLabels:
      app: <deployment-name>
  template:
    metadata:
      labels:
        app: <deployment-name>
    spec:
      containers:
        - name: nginx
          image: nginx:1.26

```
Set your default editor (e.g., nano) for updating:

```bash
export EDITOR=nano
kubectl edit deployment <deployment-name>
```
Update the image version (e.g., nginx:1.27), save, and Kubernetes will manage the rolling update. To ensure minimal downtime, you can set minReadySeconds:

```yaml
minReadySeconds: 10
```
This ensures that Kubernetes waits for 10 seconds before considering a pod as "ready."

Monitor the Update:
To check the status of the rollout:

```bash
kubectl rollout status deployment/<deployment-name>
```
## DaemonSet
A DaemonSet ensures that a specific pod runs on all nodes (or selected nodes). Here's an example:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: <daemonset-name>
  labels:
    app: <daemonset-name>
spec:
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: <daemonset-name>
  template:
    metadata:
      labels:
        app: <daemonset-name>
    spec:
      containers:
        - name: nginx
          image: nginx:1.26
```
To edit the DaemonSet, use:

```bash
kubectl edit daemonset <daemonset-name>
```
Updating the image version (e.g., nginx:1.27) will trigger a rolling update, with one pod updated at a time.

## StatefulSet
StatefulSets are used for stateful applications that require stable and unique network identities. The rolling update strategy for StatefulSets ensures that each pod is updated sequentially.You should have desired PV for running StatefulSet

Sample StatefulSet Definition:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: <statefulset-name>
  labels:
    app: <statefulset-name>
spec:
  serviceName: "<statefulset-name>-service"
  replicas: 3
  updateStrategy:
    rollingUpdate: {}
  selector:
    matchLabels:
      app: <statefulset-name>
  template:
    metadata:
      labels:
        app: <statefulset-name>
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          volumeMounts:
            - name: <volume-name>
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: <volume-name>
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi

```
Exposing the StatefulSet
You can expose the StatefulSet using a headless service or a standard service, depending on your needs.

### Persistent Volumes (PV) and Persistent Volume Claims (PVC)
Ensure that there are Persistent Volumes (PVs) available for the required storage:

```bash
kubectl get pv
```
If there are no PVs, you can create one manually, as shown below:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data  # Adjust the path based on your environment
```