## Evidently AI:
- Tool to monitor data quality, model performance and data drift.
- Monitor and evaluate machine learning models after deployment.
- Library that checks wheather your model is still working correctly in real-world data.


## Why ?
- Training data = clean, controlled
- Production data = messy, changing over time.
- So problem happens like:
    - Data drift (distribution changes over time)
    - Concept drift (relationship between features and target changes)
    - Model performance degradation (accuracy drops)

## What it does:
It continuously monitors:
```
Incoming data 
     ↓
Compare with training/reference data
     ↓
Generate reports | alerts
```
### 👉 Evidently is just a tool that tells:

“Hey, your input data changed” <br>
“Your model behavior is changing” <br>

```
User → vLLM → Output
          ↓
      Save logs
          ↓
   Check quality
          ↓
   Detect problems
```
**Evidently can be used to: It just automates this checking.** <br>

```
user
  |
istio ingress gateway
  |
fastAPI proxy  -> Logs -> storage (file/db/kafka) -> Evidently AI -> Drift / reports
  |
Models (trace-level observability)
  |
Phoenix
```
**Phoenix** -> what happens in this request? <br>
**Evidently AI** -> Is my model behaviour chnaging over time ?

## setup:
### Step 1 -- Set logging into  FastAPI Proxy `app.py`
- Logs stored into /data/llm_logs.jsonl
- /data directory is mounted on pvc `hf-cache-pvc`.

### Step 2 -- Create `evidently_config.py` and create docker image from it.
evidently_runner.py
```
"""
Evidently LLM Monitor — fixed for Evidently >= 0.7
=====================================================
Install:
    pip install "evidently>=0.7"

Run report only (no UI):
    python evidently_llm_monitor_v2.py

View in UI after running:
    evidently ui --workspace /data/evidently_workspace
    Then open http://localhost:8000
"""

import os
import pandas as pd
import evidently

from evidently import Dataset, DataDefinition, Report
from evidently.presets import DataDriftPreset

print(f"Evidently version: {evidently.__version__}")

# ─────────────────────────────────────────────
# CONFIG
# ─────────────────────────────────────────────
LOG_FILE       = "/data/llm_logs.jsonl"
WORKSPACE_PATH = "/data/evidently_workspace"
PROJECT_NAME   = "LLM Monitoring"
OUTPUT_HTML    = "/data/llm_drift_report.html"

# ─────────────────────────────────────────────
# 1. Load data
# ─────────────────────────────────────────────
df = pd.read_json(LOG_FILE, lines=True)
if df.empty:
    raise ValueError("Input log file is empty")

print(f"Loaded {len(df)} rows. Columns: {df.columns.tolist()}")

# ─────────────────────────────────────────────
# 2. Feature engineering
# ─────────────────────────────────────────────
df["prompt_len"]   = df["input"].apply(lambda x: len(str(x)))
df["response_len"] = df["output"].apply(lambda x: len(str(x)))
df["latency"]      = pd.to_numeric(df.get("latency", 0), errors="coerce").fillna(0)

# tokens is a nested dict — extract safely
if "tokens" in df.columns:
    df["total_tokens"] = df["tokens"].apply(
        lambda x: x.get("total", 0) if isinstance(x, dict) else 0
    )
else:
    df["total_tokens"] = 0

features = ["prompt_len", "response_len", "latency", "total_tokens"]
df = df[features].fillna(0)

# Replace zeros to avoid constant-column issues
df = df.replace(0, 1e-6)

# Drop constant columns (Evidently drift needs variance)
valid_cols = [c for c in df.columns if df[c].nunique() > 1]
if not valid_cols:
    raise ValueError("All columns are constant — cannot compute drift")

df = df[valid_cols]
print(f"Using features: {valid_cols}")

# ─────────────────────────────────────────────
# 3. Split reference / current
# ─────────────────────────────────────────────
split = int(len(df) * 0.7)
if split < 1 or split >= len(df):
    raise ValueError("Not enough data to split (need at least 2 rows)")

reference_df = df.iloc[:split].copy()
current_df   = df.iloc[split:].copy()
print(f"Reference rows: {len(reference_df)} | Current rows: {len(current_df)}")

# ─────────────────────────────────────────────
# 4. Wrap in Evidently Dataset  ← NEW in v0.7
#    DataDefinition replaces ColumnMapping
# ─────────────────────────────────────────────
data_def = DataDefinition(
    numerical_columns=valid_cols  # explicitly mark as numeric
)

reference_dataset = Dataset.from_pandas(reference_df, data_definition=data_def)
current_dataset   = Dataset.from_pandas(current_df,   data_definition=data_def)

# ─────────────────────────────────────────────
# 5. Build and run report  ← NEW in v0.7
#    Report([ preset ])  not  Report(metrics=[preset])
# ─────────────────────────────────────────────
report = Report([DataDriftPreset()])

# run() takes Dataset objects, not raw DataFrames
result = report.run(current_dataset, reference_dataset)

# ─────────────────────────────────────────────
# 6. Save HTML  ← use result.save_html() in v0.7
# ─────────────────────────────────────────────
result.save_html(OUTPUT_HTML)
print(f"HTML report saved → {OUTPUT_HTML}")

# ─────────────────────────────────────────────
# 7. Save to local workspace for the UI
#    workspace.add_run()  replaces  workspace.add_report()
# ─────────────────────────────────────────────
try:
    from evidently.ui.workspace import Workspace

    os.makedirs(WORKSPACE_PATH, exist_ok=True)
    workspace = Workspace.create(WORKSPACE_PATH)

    # Find or create project
    project = None
    for p in workspace.list_projects():
        if p.name == PROJECT_NAME:
            project = p
            break
    if project is None:
        project = workspace.create_project(PROJECT_NAME)
        project.description = "LLM log drift monitoring"
        project.save()

    # add_run() is the correct method in v0.7 (not add_report)
    workspace.add_run(project.id, result)
    print(f"Run logged to workspace → {WORKSPACE_PATH}")
    print(f"View in UI: evidently ui --workspace {WORKSPACE_PATH}")

except Exception as e:
    print(f"[WARN] Could not save to workspace: {e}")
    print("The HTML report was still saved successfully.")
```
Dockerfile:
```
FROM python:3.10-slim

WORKDIR /app

RUN pip install evidently pandas

COPY evidently_runner.py .

CMD ["python", "evidently_runner.py"]
```
Build and push docker image:
```
Create repository in docker.merai.app
sudo docker build -t docker.merai.app/devops/evidently:0.1 .
sudo docker push docker.merai.app/devops/evidently:0.1
```
### Step 3 -- Create `evidently_deployment_job.yaml` and deploy it.
evidently_deployment_job.yaml
```
apiVersion: batch/v1
kind: Job
metadata:
  name: evidently-job
  namespace: vllm
spec:
  template:
    spec:
      containers:
      - name: evidently
        image: docker.merai.app/devops/evidently:0.1
        imagePullPolicy: Always
        volumeMounts:
        - name: logs
          mountPath: /data
      restartPolicy: Never
      volumes:
      - name: logs
        persistentVolumeClaim:
          claimName: hf-cache-pvc
```
> log file location on host: /data/kubernetes-nfs-storage/hf-cache/llm_logs.jsonl <br>
> html file location on host : /data/kubernetes-nfs-storage/hf-cache/llm_drift_report.html

### Open the HTML file in browser:
<img width="1900" height="949" alt="image" src="https://github.com/user-attachments/assets/7b889612-f1ff-43a3-8cdb-204e58d9e139" />

