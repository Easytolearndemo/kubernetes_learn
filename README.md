=============================Day 6========================================================

## Local used in Kubernetes

kubectl - https://kubernetes.io/docs/tasks/tools/#kubectl
kind - https://kind.sigs.k8s.io/docs/user/quick-start/
Kubernetes cheat sheet : https://kubernetes.io/docs/reference/kubectl/quick-reference/

## Install Kind
```
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

## Check kind install or not
```kind --version```

## Creating a single Cluster
```kind create cluster --image kindest/node:v1.29.8@sha256:d46b7aa29567e93b27f7531d258c372e829d7224b25e3fc6ffdefed12476d3aa --name cla-cluster1```

## Install kubectl
```sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl```

## Check kubectl install or not
```kubectl version --client```

## Check how many nodes are running
```kubectl get nodes```


## Creating and configure multinode cluster
```vim config.yml```

## config.yml
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```
## Run command to apply config.yml file
```kind create cluster --image kindest/node:v1.29.8@sha256:d46b7aa29567e93b27f7531d258c372e829d7224b25e3fc6ffdefed12476d3aa --name cla-cluster2 --config config.yml```

## Check how many clusters are running
```kubectl config get-contexts```

## Switch to different cluster
```kubectl config use-context my-cluster-name```

## Delete cluster
```kind delete cluster --name cla-cluster1```

==================================Day 7=====================================================

- Imperative way create nginx pod
```kubectl run nginx-pod --image=nginx:latest```

- To know what are version(apiVersion) are support
```kubectl explain pod```

- Declarative way create nginx pod

```YAML
# This is a sample pod yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod #anything we can used
  labels: # in labels we can used anything project spacific 
    env: demo
    type: frontend
spec:
  containers:
  - name: nginx-container-test # container name for image
    image: nginx # docker image
    ports:
    - containerPort: 80 # which port is used for running nginx
```

- Create pod
```kubectl apply -f pod.yml```

- To get all pod or know which pod are running (contanaer creating -> running)
```kubectl get pods```

- Troubleshooting if pod is not create
```kubectl describe pod nginx-pod```

- know all the label particular pod
```kubectl get pods nginx-pod --show-labels```

- GO to inside container
```kubectl exec -it nginx-pod -- sh```

- know which node my pod is running
```kubectl get pods -o wide```

- know which node is running with description
```kubectl get nodes -o wide```

- Delete pod
```kubectl delete pod nginx-pod```

==================================Day 8====================================================

## Deployment 
- Deployment create -> replicaset -> then create pods


## create deployment using yml

``` YAML
apiVersion: apps/v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: demo
```


- Know how many deployment running
```kubectl get deploy```

- return all the object running in our cluster
```kubectl get all```


=======================================Day 9==============================================

## Services
There are 4 types of Services:
- ClusterIP(For Internal access)
- NodePort(To access the application on a particular port)
- LoadBalancer(To access the application on a domain name or IP address without using the port number)
- External (To use an external DNS for routing)


### Only for Kind cluster (AWS, GCP) not required
If you use a Kind cluster, if we want to perform Service means expose port then the following configuration is required. Use the below config to create a new Kind cluster

```vim cluster.yml```

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30001
    hostPort: 30001
- role: worker
- role: worker
```


### ClusterIP

#### Sample YAML for ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cluster-svc
  labels:
    env: demo
spec:
  ports:
  - port: 80
  selector:
    env: demo
```

### Nodeport

#### Sample YAML for Nodeport

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-svc
  labels:
    env: demo
spec:
  type: NodePort
  ports:
  - nodePort: 30001
    port: 80
    targetPort: 80
  selector:
    env: demo
```

### Loadbalancer

#### Sample YAML for Loadbalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-svc
  labels:
    env: demo
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    env: demo
```

- To get all service
```kubectl get svc```

- To describe particular service
```kubectl describe svc <service name>```

- Check in command service is access or not
```curl localhost:3001```

- To know everything about pod like ip, whice node running, under wich name space, etc...
```kubectl describe pod <pod name>```

- To know everything about service like port, targetPort, endpoint(here all pod ip is there, that running under that(what name we are using in command) spacific service), etc...
```kubectl describe svc <svc name>```

- To know the pod logs(if any error is comming)
```kubectl logs <pod name> -f``` 


======================================Day 10================================================

## NameSpace
- Avoid accidental deletion/modification
- Separated by resource type or environment or domain and so on
- By default creating under default namespace
- We can create same resource(pod, service, etc...) like (same name) under namespace no conflit happend



- get all namespace
```kubectl get ns```

- know which are server(pod, service, etc...) are running under namespace
```kubectl all -n <namespace-name>```

- create namespace
```kubectl create ns demo```

- create resource under the namespace
```create deploy nginx-demo --image=nginx -n demo```

- get all deploy under namespace
```get deploy -n demo```


=======================================Day 11====================================================

### Multi Container Pod Kubernetes - Sidecar vs Init Container

- After create "myservice", mydb service then only pod(myapp-pod) is create other wise pod(myapp-pod) panding state

## myservice.default.svc.cluster.local how i can get this
go to myservice pod -> ```kubectl exec -it <pod name>``` -> cat /etc/resolv.conf

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    env:
    - name: FIRSTNAME
      value: "Piyush"
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup myservice.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup mydb.default.svc.cluster.local; do echo waiting for mydb; sleep 2; done']
```

=======================================================Day 12==================================================================

### What is a daemonset?
- A daemon set is another type of Kubernetes object that controls pods. Unlike deployment, the DS automatically deploys 1 pod to each available node. You don't need to update the replica based on demand.
- If you create a DS in a cluster of 5 nodes, then 5 pods will be created.
- If you add another node to the cluster, a new pod will be automatically created on the new node.


```
apiVersion: apps/v1
kind:  DaemonSet
metadata:
  name: nginx-ds
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  selector:
    matchLabels:
      env: demo

```
==============================================Day 13===================================================

### Manual scheduling pod particular node
- Manual scheduling in Kubernetes involves assigning a pod to a specific node rather than scheduler decide.

- ```nodeName``` Field: Use this field in the pod specification to specify the node where the pod should run.

```
apiVersion: v1
kind: Pod
metadata:
  name: manual-scheduled-pod
spec:
  nodeName: worker-node-1
  containers:
  - name: nginx
    image: nginx
```

### Labels
- Labels are key-value pairs attached to Kubernetes objects like pods, services, and deployments. They help organize and group resources based on criteria that make sense to you.

- Add label in node
```kubectl label node <node name> gpu=true```

## Show all pod lebel
```kubectl get pod --show-labels```

### Selectors
- Selectors filter Kubernetes objects based on their labels. This is incredibly useful for querying and managing a subset of objects that meet specific criteria.

## Filter pod using selector
Pods: ```kubectl get pods --selector app=my-app```

### Labels vs. Namespaces 🌍
- Labels: Organize resources within the same or across namespaces.
- Namespaces: Provide a way to isolate resources from each other within a cluster.
========================================================Day 14========================================================================

![Screenshot from 2024-12-15 11-40-26](https://github.com/user-attachments/assets/872c2cb7-2bdd-4783-9f75-1abb64ddb90f)


### Taints and Tolerations
- Taints Apply -> Nodes
- Tolerations Apply -> Pods

- pods cannot be scheduled on tainted nodes unless they have a special permission called toleration. When a toleration on a pod matches with the taint on the node then only that pod will be scheduled on that node.

## Add Tainting a Node:

```bash
kubectl taint nodes node1 key=gpu:NoSchedule
```

This command taints node1 with the key "gpu" and the effect "NoSchedule." Pods without a toleration for this taint won't be scheduled there.

## To remove the taint

```bash
kubectl taint nodes node1 key=gpu:NoSchedule-
```

### Adding toleration to the pod:
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis
    name: redis
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

```

### Labels vs Taints/Tolerations
- Labels group nodes based on size, type,env, etc. Unlike taints, labels don't directly affect scheduling but are useful for organizing resources.

### Limitations to Remember 🚧
- Taints and tolerations are powerful tools, but they have limitations. They cannot handle complex expressions like "AND" or "OR." So, what do we use in that case? We use a combination of Taints, tolerance, and Node affinity, which we will discuss in the next video.

===============================================================Day 15=============================================================

### Node affinity


Specify Node Labels: Define a list of required node labels (e.g., disktype=ssd) in your pod spec.

```kubectl label node <node name> disktype=ssd```


### Properties in Node Affinity
requiredDuringSchedulingIgnoredDuringExecution - pod is sudule if label match
preferredDuringSchedulingIgnoredDuringExecution - pod issudule if label match otherwise any node



### Targeting SSD Nodes 💾
Suppose your pod needs high-speed storage. You can create a deployment with a Node Affinity rule that targets nodes labeled disktype=ssd.

YAML Configuration:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis-3
spec:
  containers:
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd

```

===========================================Day 16===================================================

### Requests and Limits

After pod alcated in nodes if we don't have no resource to alcocated new pod then we can get insuffent error in pod.


Suppose in node one pod are running, when more trafic will come then pod consume all resource(cpu, memory) from node, if we don't have resource in node for pod then we can get OOM error.

To avoid all this senaro we are using request and limit, for this node will not enter node falior insted of node pod will crase.


metrics-server.yml required apply because - it bassycally expose for node CUP, memory utilization, so we can used farther different process(Requests and Limits, autoscaling, HPA, VPA, etc...).

## metrics-server.yml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --kubelet-insecure-tls
        - --metric-resolution=15s
        image: registry.k8s.io/metrics-server/metrics-server:v0.7.1
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 10250
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100

  ```

metrics-server pod is running in kube-system name space, if we want to see then we have to run following command

```kubectl get pod -n kube-system```

If we want to see CPU memory utilization for nodes, we can see data after run following command because of metrics-server
```kubectl top nods```

mem-request.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi" # lower lemit set
      limits:
        memory: "200Mi" # upper lemit set
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"] # using this command we allocate memory (for testing purpose)

```
After apply to know how much resource consume
```kubectl top pod memory-demo -n mem-example```


```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]  # using this command we allocate memory (for testing purpose we are used 250 that overthe 100 not inbetween 50 to 100, so we can get error, OOMKilled---> error)

```


