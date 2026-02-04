---
name: devops-kubernetes
description: Kubernetes deployment, Helm charts, service mesh, and container orchestration patterns
tags: [kubernetes, k8s, helm, devops, containers]
author: Antigravity Team
version: 1.0.0
---

# Kubernetes Skill

Container orchestration with Kubernetes.

## Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: myregistry/my-app:v1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: my-app-secrets
              key: database-url
```

## Service & Ingress

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: my-app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

## ConfigMaps & Secrets

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  LOG_LEVEL: "info"
  API_TIMEOUT: "30s"
---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secrets
type: Opaque
stringData:
  database-url: "postgres://user:pass@db:5432/mydb"
```

## Helm Chart

```yaml
# Chart.yaml
apiVersion: v2
name: my-app
version: 1.0.0
appVersion: "1.0.0"

# values.yaml
replicaCount: 3
image:
  repository: myregistry/my-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  host: app.example.com

resources:
  limits:
    cpu: 500m
    memory: 128Mi
  requests:
    cpu: 250m
    memory: 64Mi
```

## Kubectl Commands

```bash
# Apply manifests
kubectl apply -f deployment.yaml

# Get resources
kubectl get pods -l app=my-app
kubectl get deployments
kubectl get services

# Debug
kubectl logs -f deployment/my-app
kubectl exec -it pod/my-app-xxx -- /bin/sh
kubectl describe pod my-app-xxx

# Scale
kubectl scale deployment my-app --replicas=5

# Rollout
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app
kubectl rollout history deployment/my-app
```

## HPA (Horizontal Pod Autoscaler)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
