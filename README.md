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

![Screenshot from 2024-12-15 15-55-55](https://github.com/user-attachments/assets/c6321366-2e25-41bb-b01e-edb63b6433d2)


Suppose in node one pod are running, when more trafic will come then pod consume all resource(cpu, memory) from node, if we don't have resource in node for pod then we can get OOM error.

![Screenshot from 2024-12-15 16-00-29](https://github.com/user-attachments/assets/4c36c7bd-4ef0-48df-a153-263bef237987)

![Screenshot from 2024-12-15 16-01-01](https://github.com/user-attachments/assets/297e0934-b796-4dcc-aef8-df924fbbf670)


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

![Screenshot from 2024-12-15 16-16-26](https://github.com/user-attachments/assets/2c5f3e21-938e-4d91-ac49-79805d2012ce)


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

=========================================Day 17==============================================================

### Kubernetes Autoscaling | HPA Vs VPA

## Autoscaling types
![Screenshot from 2024-12-15 20-38-52](https://github.com/user-attachments/assets/5e61ca55-d6fa-4617-8549-dcdc61e72de5)


## HPA v/S VPA

![image](https://github.com/user-attachments/assets/fe1dfff3-8781-459a-8828-b150a88e823b)


## Deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache

    ```

## hpa.yaml

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```


# If CPU utilization mort han 50% then pod will increase automatically (minmum pod 1 and maximum pod 10)

```kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10```

# Using this command we can generate load 
``` kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done" ```

# To know CPU utilization
```kubectl get hpa php-apache --watch```


=====================================================================Day-18==================================================================================

### Health Probes 

## What are health probes in Kubernetes?
- Health probes monitor your Kubernetes applications and take necessary actions to recover from failure, so user can not facing any problem
- To ensure your application is highly available and self-healing

## Type of health probes in Kubernetes
- Readiness (Ensure application is ready, before serving trafic to the user)
- Liveness ( monitor our application one sertan time like - every 10sec, 15sec.  Restart the application if health checks fail)
- Startup ( Probes for legacy applications that need a lot of time to start)


- First Startup Probes will active afterthat Readiness, Liveness Probes will active.

## Types of health checks they perform?
- HTTP/TCP/command


## liveness command

- In this example every 30 sec pod will fail and restart we can see using ```kubectl get po --watch``` in this command

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat 
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

```

initialDelaySeconds: 5 -> first health check happend after 5sec(why this because application take time to boot and set averything little time)
periodSeconds: 5 -> Every 5sec check application returing healthy response or not

## we can lively watch the pod
```kubectl get po --watch```

## describe pod (we can seen why pod will faild)
```kubectl describe po <pod name>```

## iveness-http and readiness-http
```
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz # inside container folder path
        port: 8080 # open port 8080 for container folder path
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe: # if readinessProbe heldhy then only then only workload will sharf
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
      ```

## Fource fully apply the pod
```kubectl apply --force -f <file name>```


## liveness-tcp


```
apiVersion: v1
kind: Pod
metadata:
  name: tcp-pod
  labels:
    app: tcp-pod
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 8000
      initialDelaySeconds: 10
      periodSeconds: 5
```


```
apiVersion: v1
kind: Pod
metadata:
  name: tcp-pod
  labels:
    app: tcp-pod
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 3000
      initialDelaySeconds: 10
      periodSeconds: 5
```

=====================================================================Day-19===================================================================================

### Config map in Kubernetes?
- When your manifest grows it becomes difficult to manage multiple env vars
- You can take this out of the manifest and store as a config map object in the key-value pair
- Then you can inject that config map into the pod
- You can reuse the same config map into multiple pods



## Createing configmap imparative way

```kubectl create cm <configmapname> --from-literal=color=blue \
--from-literal=color=red```

## Show config map
```kubectl get cm```


## declarative way config map

## cmap.yml
```
apiVersion: v1
data:
  firstname: piyush
  lastname: sachdeva
kind: ConfigMap
metadata:
  name: app-cm

  ```

## use-conf-map.yml

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
      valueFrom:
        configMapKeyRef:
          name: app-cm
          key: firstname
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    ```

### We can also used simple way
- doc
- Follow the doc: https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data

=============================================Day-20=================================================

## Suppose any new user comes as a kubarnates adminstrator, kubarnates admin need to give a permison to work

1) new used net create the certificate request.
- To generate a key file
```openssl genrsa -out adam.key 2048```

2) To generate a csr file
```openssl req -new -key adam.key -out adam.csr -subj "/CN=adam"```

csr.yml

```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: adam
spec:
  that following key that generate from new user, this suld be .csr key that user generate
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo= 
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # expiration seconds one day
  usages:
  - client auth

```
- in .csr file key is a plan text format, we have to convert base64 encoded and in a single line

```cat adam.csr | base64 | tr -d "\n"```

- From uper command we get one key, that key old kubarnates adminstrator add in csr.yml in request as a value.

- then i have to apply that csr.yml
```kubectl -f csr.yml```

- if we want to see all the certificates
```kubectl get csr```

- we can see from uper command pending status, that means kubarnates adminstrator need to approve, following command
## To approve a csr
```kubectl certificate approve <certificate-signing-request-name>```

## To deny a csr
```kubectl certificate deny <certificate-signing-request-name>```

create yml file command
```kubectl get csr adam -o yml > issuecert.yml```

# from issuecert.yml we have one request key taht value we have to copy and generate base64 encoded -> then we can share that base64 encoded key to the new user

# create base64 encoded key

copy thet request key -> ```echo "<past key>" | base64 -d``` -> enter from keybord

that generate key we have to share to the user



