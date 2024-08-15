# Helm

## What it solves?

Help maintain K8s yml files when we need to upscale

## What is Helm?

Helm is often referred to as a Kubernetes package manager because it was made for bundling a couple of Kubernetes
resource files into a single package that can be easily deployed

Main aim:

- reduce the complexity of Kubernetes resource management by providing its abstraction
- allows for the installation of applications with a single command with a default configuration

All of that is possible because chart developers are preparing templates—blueprints for all Kubernetes resource
files—making editable only those parts that are important from the application-level point of view, like how many
instances of services need to be deployed or what environment variables are injected into running containers. A chart
takes the burden of managing Kubernetes’s specific API and allows it to focus on more high-level tasks

Sample helm chart file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
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
          image: 'nginx:{{ .Values.container.image.version | default "1.21" | quote }}'
          ports:
            - containerPort: 80
```

{{}} -> place holder telling Helm that a particular part is not static and requires some actions to be performed

2 ways to override the value:

- use set option
- -f with values.yaml file

Each time use 'helm install' or 'helm upgrade' -> create new release + save history -> easily roll back entire stack

## Key benefits

Benefit:

- Parameterized Kubernetes YAMLs
- Reduces the complexity of installing an application on Kubernetes: It can be achieved using a single command.
- Keeps track of the history of revisions
