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

=======================================================================================

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

======================================================================================

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


=====================================================================================

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


======================================================================================

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


===========================================================================================
