# Volume

## State preservation

Most of the time, stateful applications store their state on disk. This leaves us with a problem. If a container
crashes, kubelet will restart it. The problem is that it will create a new container based on the same image. All data
accumulated inside a container that crashed will be lost

## K8s Volume

Kubernetes volumes solve the need to preserve the state across container crashes. In essence, volumes are references to
files and directories that are accessible to containers that form a Pod

Diff types of K8s volume -> the way file/dir created

## Hands-on

Define volumeMounts:

- mountPath: Where we expect to mount INSIDE THE CONTAINER
- name: name of the volume

Define the volume:

The hostPath volume allows us to mount a file or a directory from a host to Pods and, through them, to containers

Note: Do not use hostPath to store a state of an application. Since it mounts a file or a directory from a host into a
Pod, it is not fault-tolerant. If the server fails, Kubernetes will schedule the Pod to a healthy node, and the state
will be lost

