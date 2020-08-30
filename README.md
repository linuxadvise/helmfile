## Chart Description 

#### Chart tree

````
.
├── charts
│   └── appname
│       ├── charts
│       ├── Chart.yaml
│       ├── templates
│       │   ├── deployment.yaml
│       │   ├── _helpers.tpl
│       │   ├── ingress.yaml
│       │   ├── NOTES.txt
│       │   ├── serviceaccount.yaml
│       │   ├── service.yaml
│       │   └── tests
│       │       └── test-connection.yaml
│       └── values.yaml
├── helmfile.d
│   ├── helmfile.yaml
│   ├── releases
│   │   └── appname.yaml
│   └── values
│       └── appname.yaml.gotmpl
├── README.md
````

> Main folder with templating chart located at `helm/charts`

### Description 

The most interesting thing is located here `helm/helmfile.d`

`helmfile.yaml` - this file contains path to relises file

```kubernetes helm
helmfiles:
  - "releases/appname.yaml"
```

`releases/appname.yaml` - this file contains main parameters for release 

```kubernetes helm
releases:
  # Name for release
  - name: "appname"
    # Namespace for application 
    namespace: {{ env "NAMESPACE" | default "appnamespace" }}
    labels:
      # Chart name
      chart: "appname"
      # Component for chart
      component: "appname"
      # Namespace for application 
      namespace: {{ env "NAMESPACE" | default "appnamespace" }}
    # Path to the chart
    chart: "../../charts/appname"
    # Options - wait when release deployed
    wait: true
    # set `false` to uninstall this release on sync.  (default true)
    installed: {{ env "INSTALLED" | default "true" }}
    # Path to variables files
    values:
    - ../values/appname.yaml.gotmpl
```

`values/appname.yaml.gotmpl` - this file contains variables for chart

```kubernetes helm
image:
  # Environment variable with contain url for docker image in gcr
  repository: {{ requiredEnv "REPOSITORY_URL" }}
  # Tag for docker image
  tag: {{ requiredEnv "IMAGE_TAG" }}

# Enable service creating
service:
  enabled: true


ingress:
  # Enable ingress creating
  enabled: true
  annotations:
    # Choose ingress controller 
    kubernetes.io/ingress.class: "nginx"
    # Enable ssl redirect 
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    # Choose cluster-issuer
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    # Domain name for http
    - host: {{ requiredEnv "DOMAIN_NAME" }}
    # Default path 
      paths:
        - /
  tls:
  - hosts:
    # Domain name for https
    - {{ requiredEnv "DOMAIN_NAME" }}
    # Secret name which contains ssl certificate
    secretName: {{ requiredEnv "DOMAIN_NAME" }}-tls-cert


resources:
  limits:
  # CPU limit
    cpu: 500m
  # Memory limit
    memory: 512Mi
  requests:
  # Default CPU 
    cpu: 200m
  # Default memory
    memory: 256Mi

ports:
  - name: http
    # Port for application container
    containerPort: 3000
    protocol: TCP

livenessProbe:
  initialDelaySeconds: 30
  tcpSocket:
    port: 3000
readinessProbe:
  initialDelaySeconds: 30
  tcpSocket:
    port: 3000

# Envirinment variables for application container
env:
  - name: NODE_ENV
    value: "development"
  - name: DB
    value: {{ env "DB" }}
```

### Adding environment variables

In order to add secret environment variables, you need to do the following. 
Go to your project in gitlab - `settings` - `CI/CD` - `Variables` - click `Add variable` choose `environment name` and add your variable `name` and `value` 

