# Deployments

## Purpose

Useful when deploy a new release to a cluster
Must provide zero downtime

## Process breakdown

1. K8s client call API server to request creation
2. Deployment controller watch for new events -> detect new Deployment object
3. Deployment controller create new ReplicaSet object

## Explain how it works

Instead of operating directly on the level of Pods, the Deployment
creates a new ReplicaSet which subsequently produces Pods based on the new image.
Once they become fully operational, the Deployment scales the old ReplicaSet to 0

## Update Deployments

### Update by cmd

```shell
kubectl edit -f go-demo-2-db.yml
```

Not recommend

### Update yml file

Update the YAML file and execute the kubectl apply command

Not fit for applications that change weekly/daily/hourly

### Defining zero-downtime deployment

1. minReadySeconds: defines the minimum number of seconds
   before Kubernetes starts considering the Pods healthy. Default 0
2. revisionHistoryLimit: defines the number of old ReplicaSets we can roll back
3. strategy: RollingUpdate or Recreate.
   Recreate -> stop existing one then create new one -> downtime
   -> Fit if application should not have 2 releases at 1 time
   -> Suit for single-replica DB. Need to mount volume for data persistent
   RollingUpdate -> Create new ReplicasSet, increase new one then decrease old one
   maxSurge -> maximum nums of Pods that exceeds desired number. default 25%
   maxUnavailable -> maximum numbs of Pods can't be operated
4. template

### Creating zero-downtime deployment

### Rolling back/forward

### Update multiple deployments

Almost everything in Kubernetes is operated using label selectors
The trick is to find a label (or a set of labels) that uniquely identifies all the Deployments we want to update

### Scaling/Updating deployments

#### Using YAML

If we decide that the number of replicas changes with relatively low frequency or that Deployments are performed
manually, the best way to scale is to write a new YAML file or, even better, modify the existing one. Assuming that we
store YAML files in a code repository, by updating existing files we have a documented and reproducible definition of
the objects running inside a cluster

Please note that, even though the file is different, the names of the resources are the same so kubectl apply did not
create new objects. Instead, it updated those that changed

### Conclusion

![img.png](img/Deployments%20Overview.png)