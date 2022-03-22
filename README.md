# k8S

## Namespace
isolating groups of resources within a single cluster.
Names of resources need to be unique within a namespace, but not across namespaces
Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, pods, replica set)

below command returns available namespace in the kubernetes cluster
`kubectl get namespace`

Kubernetes starts with four initial namespaces:
 - `default` The default namespace for objects with no other namespace
 - `kube-system` The namespace for objects created by the Kubernetes system
 - `kube-public` This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
 - `kube-node-lease` This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.


### Create new namespace
#### using command:
`kubectl create namespace <namespace_name>` ex: `kubectl create namespace seetha`

kubectl create namespace pap
kubectl create namespace capstone
kubectl create namespace Proj_name

#### using file:
create yaml file --> namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
```

`kubectl apply -f namespace.yaml` --> will create namespace

### Delete namespace
`kubectl delete namespace <namespace_name>`

## Pods
A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

to get all pods
`kubectl get pod` --> returns pod in a current namespace
`kubectl get pod -a` --> returns pod from all namespace


pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx_example
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
`kubectl apply -f pod.yaml`


### Init containers
specialized containers that run before app containers in a Pod. 
Init containers can contain utilities or setup scripts not present in an app image.
 - Init containers always run to completion.
 - Each init container must complete successfully before the next one starts.

If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds.

Example
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

## Replica Set
A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: demo
    tier: frontend
spec:
  replicas: 3
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
        image: nginx:1.14.2
```

`kubectl get rs`
`kubectl describe rs/frontend`

`kubectl get pods` --> you can see pods which are created from above


## Deployment
A Deployment provides declarative updates for Pods and ReplicaSets.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```

### Rollout and rollback history (todo)

### log into the container 
`kubectl exec --stdin --tty <pod_name> -- /bin/bash`

### Run individual command in a container
`kubectl exec <pod_name> -- ls /`

### Set environment variable
```
apiVersion: v1
kind: Pod
metadata:
  name: env-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
      - name: DB_USER_NAME
        value: db_user
      - name: DB_USER_PASSWORD
        value: db_password
      - name: DB_HOST
        value: db_host
```

### ConfigMaps
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

#### configmap file example:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-configmap
data:
  # property-like keys; each key maps to a simple value
  id: "3"
  name: "seetha"
```

#### use it in pods
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: USER_NAME # Notice that the case is different here
          valueFrom:
            configMapKeyRef:
              name: app-configmap           # The ConfigMap this value comes from.
              key: name # The key to fetch.
        - name: USER_ID
          valueFrom:
            configMapKeyRef:
              name: app-configmap
              key: id
```


### Secrets
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code.

https://kubernetes.io/docs/concepts/configuration/secret/

https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/


```
apiVersion: v1
kind: Secret
metadata:
  name: secret-demo
type: Opaque
data:
  username: c2VldGhh
  password: c2VldGhhMTIz
```

example of using secrets in pods
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
```

# Replica Set Name: <deploy_name>_random
# Pods Name: <replica_name>_random


# Services
- minikube service <svc_name>

## NodePort


## ClusterIp

# created service
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    run: nginx-svc
spec:
  # type: NodePort
  # type: ClusterIP 
  ports:
  - port: 8080 # nginx-svc:8080 --> nginx:80
    targetPort: 80 # app container's port
    # nodePort: 30000 # localhost/mikube_ip:30000 30000-32767, Optional field
    protocol: TCP
    name: http
  selector:
    app: nginx-app
```

### make new pod 
```
apiVersion: v1
kind: Pod
metadata:
  name: curl
spec:
  containers:
  - name: curl
    image: radial/busyboxplus:curl
    command: ["sh", "-c", "sleep 300"]
```
### login to curl pod

$ kubectl exec --stdin --tty curl  -- /bin/sh

### test app
$ curl nginx-svc:8080


### Volumes

#### Empty Dir


#### Persistent Volume

1. PersistentVolume ( Disk space available for use )
2. You define a PersistentVolumeClaim - you claim usage of a part of that PersistentVolume disk space.
3. You create a Pod that refers to your PersistentVolumeClaim