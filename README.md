# Kubernetes Cheat Sheet

## Common Kubernetes Commands

### 1. Cluster Information
```bash
# Get cluster information
kubectl cluster-info
```

### 2. Working with Namespaces
```bash
# List all namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace <namespace-name>

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace-name>
```

### 3. Managing Pods
```bash
# List all pods in current namespace
kubectl get pods

# List all pods across all namespaces
kubectl get pods --all-namespaces

# Describe a specific pod
kubectl describe pod <pod-name>

# Delete a pod
kubectl delete pod <pod-name>

# Tail pod logs
kubectl logs -f <pod-name>

# Execute a command in a running pod
kubectl exec -it <pod-name> -- /bin/bash
```

### 4. Managing Deployments
```bash
# List all deployments
kubectl get deployments

# Create a deployment
kubectl create deployment <deployment-name> --image=<image-name>

# Scale a deployment
kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>

# Update a deployment's image
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Rollback a deployment to the previous revision
kubectl rollout undo deployment/<deployment-name>

# View deployment status
kubectl rollout status deployment/<deployment-name>
```

### 5. Managing Services
```bash
# List all services
kubectl get services

# Expose a deployment as a service
kubectl expose deployment <deployment-name> --type=<ClusterIP|NodePort|LoadBalancer> --port=<service-port>

# Get details of a specific service
kubectl describe service <service-name>
```

### 6. Managing ConfigMaps
```bash
# Create a ConfigMap from a file
kubectl create configmap <configmap-name> --from-file=<path-to-file>

# Create a ConfigMap from literal key-value pairs
kubectl create configmap <configmap-name> --from-literal=<key>=<value>

# Get all ConfigMaps
kubectl get configmaps

# Describe a specific ConfigMap
kubectl describe configmap <configmap-name>
```

### 7. Managing Secrets
```bash
# Create a Secret from literal values
kubectl create secret generic <secret-name> --from-literal=<key>=<value>

# Create a Secret from a file
kubectl create secret generic <secret-name> --from-file=<path-to-file>

# List all secrets
kubectl get secrets

# Describe a specific Secret
kubectl describe secret <secret-name>
```

### 8. Working with Nodes
```bash
# List all nodes
kubectl get nodes

# Describe a specific node
kubectl describe node <node-name>

# Cordon a node (mark as unschedulable)
kubectl cordon <node-name>

# Drain a node (evict all pods from node)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon a node (make schedulable again)
kubectl uncordon <node-name>
```

### 9. Apply Configuration from YAML
```bash
# Apply a configuration file
kubectl apply -f <file.yaml>

# Delete resources defined in a YAML file
kubectl delete -f <file.yaml>
```

### 10. Checking Resource Usage
```bash
# View resource utilization of nodes and pods
kubectl top nodes
kubectl top pods
```

## Common Kubernetes Configuration Files
### 1. Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### 2. Service (ClusterIP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### 3. Service (NodePort)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```

### 4. Service (LoadBalancer)
A LoadBalancer service for exposing your application externally.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 5. Ingress
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### 6. ConfigMap
A ConfigMap stores configuration data in key-value pairs.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  configKey1: configValue1
  configKey2: configValue2
```

### 7. Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # Base64 encoded
  password: MWYyZDFlMmU2N2Rm  # Base64 encoded

```

### 8. Persistent Volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
```

### 9. Persistent Volume Claim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```


### 10. Horizontal Pod Autoscaler
A HorizontalPodAutoscaler automatically scales the number of pods in a deployment.
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### 11. Job
A Job creates one or more pods and ensures that a specified number of them successfully terminate.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: my-container
        image: busybox
        command: ["echo", "Hello Kubernetes"]
      restartPolicy: Never
  backoffLimit: 4
```

### 12. CronJob
A CronJob runs jobs on a time-based schedule.
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"  # Runs every 5 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: my-container
            image: busybox
            command: ["echo", "Hello from CronJob"]
          restartPolicy: OnFailure
```
