# MlFlow
```
Github + Experiment tracer + Model registry for ML
```

### Without MLFlow:
Noobody knows:
- Which model performed best?
- Which dataset was used ?
- which hyperparameters worked ?
- Which model is in production ?

These becomes messy. <br>

It helps track:
- Experiments
- Parameters
- Versions
- Metrics
- Models
- deployments

## Core Components
- Tracking
- Model Registry
- Artifacts
- Deployments



Implemented project - https://github.com/purvalpatel/Sample-mlops-project <br>

- Experiment tracking + Model Registry.
- Log metrics
- Log Parameters
- Version models
- Register models 
- Deploy models

```
Try 10 models -> Compare -> Pick best -> Deploy
```
### Production stack:

- MLflow + Kubernetes : scalable deployment
- MLflow + GPUs :	deep learning
- MLflow + DVC	: dataset versioning
- MLflow + ArgoCD	: GitOps ML
- MLflow + vLLM :	LLM serving
- MLflow + LoRA	: fine-tuning tracking

## Deployment MLFLow

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
