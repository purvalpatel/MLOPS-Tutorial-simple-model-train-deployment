Implemented project - https://github.com/purvalpatel/Sample-mlops-project

- Experiment tracking + Model Registry.
- Log metrics
- Log Parameters
- Version models
- Register models 
- Deploy models

```
Try 10 models -> Compare -> Pick best -> Deploy
```

Deployment `mlflow-deployment.yaml`
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mlflow
  namespace: mlflow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mlflow
  template:
    metadata:
      labels:
        app: mlflow
    spec:
      containers:
        - name: mlflow
          image: ghcr.io/mlflow/mlflow:latest
          args:
            [
              "mlflow",
              "server",
              "--host",
              "0.0.0.0",
              "--port",
              "5000",
              "--default-artifact-root",
              "file:/mlflow/artifacts"
            ]
          ports:
            - containerPort: 5000
          volumeMounts:
            - name: mlflow-artifacts
              mountPath: /mlflow/artifacts
      volumes:
        - name: mlflow-artifacts
          persistentVolumeClaim:
            claimName: mlflow-pvc

```

PVC `mlflow-pvc.yaml`
```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mlflow-pvc
  namespace: mlflow
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: k8s-nfs-storage
  resources:
    requests:
      storage: 1Gi
```

Service `mlflow-service.yaml`
```YAML
apiVersion: v1
kind: Service
metadata:
  name: mlflow-service
  namespace: mlflow
spec:
  type: NodePort
  selector:
    app: mlflow
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 31555
```
