### Namespace

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: bar
```

### Deployment

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: bar
  labels:
    app: nginx
spec:
  replicas: 1
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
          image: aputra/nginx:v0.1.3
          ports:
            - containerPort: 80
```

### Secrets

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: lesson-158-private
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  url: git@github.com:antonputra/lesson-158-private.git
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAt
    QyNTUxOQAAACAvG2MZK7C4yJzkfQcunxQp2OpOluYI+pSJmdfL4HNrwwAAAJine2Aip3tg
    IgAAAAtzc2gtZWQyNTUxOQAAACAvG2MZK7C4yJzkfQcunxQp2OpOluYI+pSJmd
    AAAEApkokq4ThivZHCdSZE+xQBI/DvJki6B7QhPQUpGfzTTS8bYxkrsLjInOR9By6fFCnY
    6k6W5gj6lImZ18vgc2vDAAAAFWFyZ29jZEBhbnRvbnB1dHJhLmNvbQ==
    -----END OPENSSH PRIVATE KEY-----
  insecure: "false"
  enableLfs: "true"
```

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub
  namespace: argocd
type: Opaque
stringData:
  my-token: aputra:dckr_pat_LeWZtI5IEfgBjRcSVx8uT8prgKU
```

### Service

```yaml
---
apiVersion: v1
kind: Service
metadata:
  namespace: monitoring
  name: grafana
spec:
  type: ClusterIP
  ports:
    - name: service
      port: 3000
      targetPort: grafana
      protocol: TCP
  selector:
    app: grafana
```

### configmap

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: datasources
  namespace: monitoring
data:
  datasources.yaml: |
    apiVersion: 1
    datasources:
    - name: Main
      type: prometheus
      url: http://prometheus-operated.monitoring:9090
      isDefault: true
      jsonData:
        manageAlerts: false
```

### Deployment

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: aputra/myapp-182:0.1.0
          ports:
            - containerPort: 8080
```

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: aputra/myapp-182:0.1.0
          ports:
            - name: http
              containerPort: 8080
          readinessProbe:
            httpGet:
              path: /status
              port: http
            initialDelaySeconds: 10
            periodSeconds: 3
```

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: aputra/myapp-182:0.1.0
          ports:
            - name: http
              containerPort: 8080
          readinessProbe:
            httpGet:
              path: /status
              port: http
            initialDelaySeconds: 10
            periodSeconds: 3
```

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:9.4.1
          ports:
            - containerPort: 3000
          volumeMounts:
            - name: storage
              mountPath: "/var/lib/grafana"
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: grafana
```

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: aputra/myapp-182:0.1.0
          ports:
            - containerPort: 8080
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - myapp
              topologyKey: "kubernetes.io/hostname"
```

### Statefulset

```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp
spec:
  replicas: 2
  serviceName: myapp
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
          image: nginx:1.25.3
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

### Daemonset

```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
spec:
  selector:
    matchLabels:
      name: fluent-bit
  template:
    metadata:
      labels:
        name: fluent-bit
    spec:
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:2.1.9
      tolerations:
        - key: instance_type
          value: gpu
          effect: NoSchedule
          operator: Equal
        - key: instance_type
          value: spot
          effect: NoSchedule
          operator: Equal
```

### Pod

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: migration
spec:
  containers:
    - name: migration
      image: ubuntu:22.04
      command: ["echo", "Run migration"]
```

### Job

```yaml
---
apiVersion: batch/v1
kind: Job
metadata:
  name: migration
spec:
  backoffLimit: 2
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migration
          image: ubuntu:22.04
          command: ["echo", "Run migration"]
```

### InitContainers and Sidecars

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  initContainers:
    - name: init-repo
      image: busybox:1.28
      command: ["sh", "-c", "echo clone repo ..."]
      volumeMounts:
        - mountPath: /repo
          name: repo
  containers:
    - name: myapp
      image: aputra/myapp-182:0.1.0
      ports:
        - containerPort: 8080
      volumeMounts:
        - mountPath: /repo
          name: repo
  volumes:
    - name: repo
      emptyDir:
        sizeLimit: 1Gi
```