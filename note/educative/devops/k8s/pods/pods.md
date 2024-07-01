# K8S Pods

## Pods

- Smallest unit of K8s
- Contains 1 or more container. Everything in pod is tightly coupled

### Pods wrapper

Recommend: single container in 1 pod. Can't tell k8s to run a container -> wrap into pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    type: db
    vendor: MongoLabs
spec:
  containers:
    - name: db
      image: mongo:3.3
      command: [ "mongod" ]
      args: [ "--rest", "--httpinterface" ]
```

- apiVersion + kind -> mandatory
- metadata: define pod metadata (name, labels)
- spec

### Pod scheduling step-by-step

![img.png](image/Pod%20scheduling%20sequence.png)

### Working with pods

Kubernetes guarantees that the containers inside a Pod are (almost) always running

#### Pod shutdown process

The first thing it does is to send the TERM (terminate) signal to all the main processes inside the containers that form
the Pod.

From there on, Kubernetes gives each container a period of thirty seconds so that the processes in those containers can
shut down properly.

Once the grace period expires, the KILL signal is sent to terminate all the main processes forcefully and, with them,
all the containers. The default grace period can be changed through the gracePeriodSeconds value in the YAML definition
or --grace-period argument of the kubectl delete command.

### Multiple container in a single pod

Pods are designed to run multiple cooperative processes that should act as a cohesive unit. Those processes are wrapped
in containers

All the containers that form a Pod are running on the same machine. A Pod cannot be split across multiple nodes

All the processes (containers) inside a Pod share the same set of resources, and they can communicate with each other
through localhost. One of those shared resources is storage

Sample:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: go-demo-2
  labels:
    type: stack
spec:
  containers:
    - name: db
      image: mongo:3.3
    - name: api-1
      image: vfarcic/go-demo-2
      env:
        - name: DB
          value: localhost
    - name: api-2
      image: vfarcic/go-demo-2
      env:
        - name: DB
          value: localhost
```

### Compare single-container pod to multiple-container pods

We should not think of Pods as resources that should do anything beyond the definition of the smallest unit in our
cluster. A Pod is a collection of containers that share the same resources. Not much more. Everything else should be
accomplished with higher-level constructs

### Pods healthcheck

Sample 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: go-demo-2
  labels:
    type: stack
spec:
  containers:
    - name: db
      image: mongo:3.3
    - name: api
      image: vfarcic/go-demo-2
      env:
        - name: DB
          value: localhost
      livenessProbe:
        httpGet:
          path: /this/path/does/not/exist
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 2 # Defaults to 1
        periodSeconds: 5 # Defaults to 10
        failureThreshold: 1 # Defaults to 3
```
