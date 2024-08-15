# Create a custom helm chart

## Creating a blueprint

Command to create new blueprint app

```shell
helm create APP_NAME
```

App generated structure

```text
────app
    │   .helmignore
    │   Chart.yaml
    │   values.yaml
    │
    ├───charts
    └───templates
        │   deployment.yaml
        │   hpa.yaml
        │   ingress.yaml
        │   NOTES.txt
        │   service.yaml
        │   serviceaccount.yaml
        │   _helpers.tpl
        │
        └───tests
                test-connection.yaml
```

Explain:

- .helmignore: specify which files will be ignored during the packaging of a Helm chart
- Chart.yaml: hold chart info (name, author, version, or dependencies)
- values.yaml: holds information about the default configuration for this chart
- templates: Helm template files—blueprints for Kubernetes resources with supporting files

## Define kubernetes object

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.app.name }}
  labels:
    app.kubernetes.io/version: {{ .Release.Name }}-{{ .Release.Revision }}
```

.Values : Helm built-in object that specifies that an actual value is in values.yaml

app.name: tells the location from where the value should be taken

.Release: include the name of the release and its version

## Define a list of env var

Use range function for env vars

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: { { .Values.app.name } }
  labels:
    app.kubernetes.io/name: { { .Values.app.name } }
    app.kubernetes.io/version: { { .Release.Name } }-{{ .Release.Revision }}
    app: { { .Values.app.name } }
    group: { { .Values.app.group } }
spec:
  replicas: { { .Values.app.replicaCount } }
  selector:
    matchLabels:
      app: { { .Values.app.name } }
  template:
    metadata:
      labels:
        app: { { .Values.app.name } }
        group: { { .Values.app.group } }
    spec:
      containers:
        - name: { { .Values.app.name } }
          image: { { .Values.app.container.image } }
          ports:
            - containerPort: { { .Values.app.container.port } }
          env:
            { { - range .Values.app.container.env } }
            - name: { { .key } }
              value: { { .value } }
            { { - end } }
```

