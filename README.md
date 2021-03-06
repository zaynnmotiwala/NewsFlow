# NewsFlow — News Classification MLOps Platform

An end-to-end Machine Learning Operations (MLOps) project for automated news classification, reproducible model training, experiment tracking, API-based inference, and model monitoring.

**Reproducible ML Pipelines · Experiment Tracking · Automated Testing · CI/CD · Docker · Model Monitoring**

---

## Overview

NewsFlow demonstrates how a machine learning application can move beyond experimentation and become a structured, deployable system.

The project uses the **AG News dataset** to classify news articles into four categories:

- World
- Sports
- Business
- Sci/Tech

The primary objective is to demonstrate the engineering practices involved in building and maintaining a machine learning pipeline, rather than focusing solely on model development.

### Key Features

| Feature | Description |
|---|---|
| Data Processing | Automated ingestion, validation, and preprocessing |
| Experiment Tracking | MLflow-based experiment tracking and comparison |
| Data Versioning | DVC pipeline and dataset versioning |
| Model Selection | Validation-based model selection and promotion |
| Model Serving | FastAPI REST API for predictions |
| Batch Inference | Classification of multiple articles in one request |
| Automated Testing | Test coverage across data, model, and API components |
| CI/CD | GitHub Actions pipeline for automated validation and deployment |
| Drift Monitoring | Evidently AI-based data distribution monitoring |
| Containerization | Docker-based application deployment |

---

## Architecture

The application is organized into several interconnected stages:

1. **Data Layer:** Downloads and validates the AG News dataset.
2. **Data Versioning:** Uses DVC to track datasets and pipeline stages.
3. **Experiment Tracking:** Records model parameters, metrics, and artifacts using MLflow.
4. **Model Registry:** Registers and promotes the selected model based on validation performance.
5. **Testing:** Runs automated checks against data processing, model behavior, and API functionality.
6. **CI/CD:** Automates testing, training, evaluation, and Docker image creation.
7. **Model Serving:** Exposes trained models through a FastAPI application.
8. **Monitoring:** Compares incoming data distributions against reference data.

### Architecture Flow

```text
             AG News Dataset
                    |
                    v
             Data Ingestion
                    |
                    v
          Data Validation
                    |
                    v
            Preprocessing
                    |
                    v
             DVC Pipeline
                    |
                    v
          Model Experiments
                    |
                    v
             MLflow
                    |
                    v
           Model Registry
                    |
                    v
          Model Evaluation
                    |
                    v
             FastAPI
                    |
                    v
          Prediction API
                    |
                    v
          Drift Monitoring
```

---

## Pipeline Components

### 1. Data Ingestion and Validation

The ingestion component downloads the AG News dataset and performs initial data quality checks.

Implemented checks include:

- Required column validation.
- Null-value ratio checks.
- Class distribution checks.
- Duplicate text detection.

The purpose is to identify data quality problems before model training begins.

### 2. Text Preprocessing

The preprocessing component cleans the text and creates stratified training, validation, and testing datasets.

The documented split is:

| Dataset | Percentage |
|---|---:|
| Training | 70% |
| Validation | 15% |
| Testing | 15% |

The generated CSV files are tracked through DVC.

### 3. Experiment Tracking

The training component compares three configurations using TF-IDF feature extraction and different classification algorithms.

| Experiment | Model | N-gram Range |
|---|---|---|
| Baseline | Logistic Regression | (1, 1) |
| Bigrams | Logistic Regression | (1, 2) |
| SVM Bigrams | Calibrated SVM | (1, 2) |

MLflow records:

- Hyperparameters.
- Accuracy and F1 scores.
- Precision and recall.
- AUC-ROC.
- Model artifacts.
- Confusion matrices.
- Classification reports.

### 4. Model Selection and Promotion

The best-performing model is selected using validation accuracy.

The MLflow Model Registry manages model versions and promotion between Staging and Production.

The documented default accuracy threshold is **0.87**, configured in `params.yaml`.

### 5. Automated Testing

The project documents three test modules:

| Test Module | Coverage | Test Count |
|---|---|---:|
| `test_data.py` | Schema, data types, split integrity, and data leakage | 21 |
| `test_model.py` | Model loading, predictions, output shapes, and probability checks | 21 |
| `test_api.py` | API endpoints, validation, edge cases, and batch inference | 29 |
| **Total** | | **71** |

Run the tests using:

```bash
pytest tests/ -v --tb=short
```

The original project documentation reports 71 passing tests. Run the suite locally to confirm the current result.

### 6. Continuous Integration and Deployment

GitHub Actions automates the documented pipeline through four jobs:

1. Linting and testing.
2. Model training and evaluation.
3. Docker image building and publishing.
4. Pipeline execution summary.

The training stage includes an accuracy gate before proceeding to the deployment stage.

The Docker workflow also includes a Trivy security scan.

### 7. Model Serving

The FastAPI application exposes endpoints for individual predictions, batch predictions, model information, and health checks.

The serving component loads the Production model from MLflow, with a local model-file fallback documented for tracking-server availability issues.

### 8. Drift Monitoring

The monitoring component uses Evidently AI to compare incoming text data with the training reference dataset.

The documented monitoring features include:

- Text length.
- Word count.
- Average word length.
- Number of sentences.
- Uppercase ratio.
- Digit ratio.

Population Stability Index (PSI) is used to evaluate distribution changes. An HTML monitoring report is generated at:

```text
reports/drift_report.html
```

---

## Project Structure

```text
NewsFlow/
│
├── data/
│   ├── raw/
│   └── processed/
│       ├── train.csv
│       ├── val.csv
│       └── test.csv
│
├── src/
│   ├── ingest.py
│   ├── preprocess.py
│   ├── train.py
│   ├── evaluate.py
│   └── serve.py
│
├── tests/
│   ├── test_data.py
│   ├── test_model.py
│   └── test_api.py
│
├── monitoring/
│   └── monitor.py
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── models/
├── reports/
├── mlruns/
│
├── Dockerfile
├── docker-compose.yml
├── dvc.yaml
├── params.yaml
├── pytest.ini
└── requirements.txt
```

---

## Installation and Setup

### Prerequisites

Install the following tools before starting:

- Python 3.10 or later
- Git
- Docker
- Docker Compose
- pip

### 1. Clone the Repository

Replace the repository URL with your own repository URL.

```bash
git clone https://github.com/YOUR_USERNAME/NewsFlow.git
cd NewsFlow
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Initialize DVC

If DVC has not already been initialized in the repository:

```bash
dvc init
```

To track the raw dataset directory:

```bash
dvc add data/raw
```

Then commit the generated DVC tracking files:

```bash
git add data/raw.dvc .gitignore
git commit -m "chore: initialize data versioning"
```

### 4. Start the MLflow Tracking Server

```bash
mlflow server \
  --host 0.0.0.0 \
  --port 5001 \
  --backend-store-uri sqlite:///mlruns/mlflow.db \
  --default-artifact-root ./mlruns/artifacts
```

The MLflow interface will be available at:

`http://localhost:5001`

### 5. Execute the Pipeline

You can run the individual stages:

```bash
python src/ingest.py
python src/preprocess.py
python src/train.py
python src/evaluate.py
```

Alternatively, execute the DVC pipeline:

```bash
dvc repro
```

### 6. Run Automated Tests

```bash
pytest tests/ -v --tb=short
```

### 7. Start the API

```bash
uvicorn src.serve:app --host 0.0.0.0 --port 8000 --reload
```

API documentation:

`http://localhost:8000/docs`

### 8. Generate a Drift Report

```bash
python monitoring/monitor.py
```

The generated report is documented at:

```text
reports/drift_report.html
```

---

## API Reference

The application provides four documented REST endpoints.

### POST `/predict`

Classifies an individual news article.

**Request:**

```json
{
  "text": "Scientists discover a new planet in the habitable zone",
  "top_k": 3
}
```

**Example response:**

```json
{
  "label": "Sci/Tech",
  "label_id": 3,
  "confidence": 0.9142,
  "top_predictions": [
    {
      "label": "Sci/Tech",
      "probability": 0.9142
    },
    {
      "label": "World",
      "probability": 0.0521
    },
    {
      "label": "Business",
      "probability": 0.0337
    }
  ],
  "model_version": "local:tfidf_svm_bigrams",
  "latency_ms": 8.4
}
```

### POST `/predict/batch`

Processes multiple articles in one request.

**Request:**

```json
{
  "texts": [
    "The central bank announces new interest rates",
    "A football team wins the championship"
  ]
}
```

### GET `/model/info`

Returns model metadata, including the model name, version, stage, and training metrics.

### GET `/health`

Checks whether the API is running and the model is loaded.

---

## Model Results

The original project documentation reports the following experiment results.

### Experiment Comparison

| Experiment | Model | Validation Accuracy | Validation F1 |
|---|---|---:|---:|
| Baseline | Logistic Regression | 0.8861 | 0.8853 |
| Bigrams | Logistic Regression | 0.8891 | 0.8885 |
| SVM Bigrams | Calibrated SVM | 0.8903 | 0.8898 |

### Final Evaluation

| Dataset Split | Accuracy | F1-macro | AUC-ROC |
|---|---:|---:|---:|
| Validation | 0.8903 | 0.8898 | — |
| Held-out Test | 0.8788 | 0.8785 | 0.9729 |

The documentation also reports approximately 91–92% test accuracy when running the pipeline on the full 120,000-sample dataset.

**Note:** These figures are documented project results, not independently reproduced benchmarks. Actual results may vary depending on the dataset, configuration, and execution environment.

---

## Configuration

The `params.yaml` file centralizes the main pipeline settings.

Example:

```yaml
data:
  dataset: "ag_news"
  max_samples: 10000

training:
  C: 1.0
  max_iter: 1000

mlflow:
  tracking_uri: "http://localhost:5001"
  accuracy_threshold: 0.87
```

Changes to configuration values can be followed by another pipeline execution using:

```bash
dvc repro
```

---

## Docker Deployment

The project includes a multi-stage Docker build for packaging the API.

Build the image locally:

```bash
docker build -t newsflow-api .
```

Run the API container:

```bash
docker run -p 8000:8000 newsflow-api
```

To start the documented services through Docker Compose:

```bash
docker-compose up mlflow api
```

Run the training job:

```bash
docker-compose --profile train up train
```

Run the monitoring job:

```bash
docker-compose --profile monitor up monitor
```

---

## Technology Stack

| Category | Technology |
|---|---|
| Programming Language | Python |
| Dataset | HuggingFace AG News |
| Machine Learning | Scikit-learn |
| Feature Extraction | TF-IDF |
| Experiment Tracking | MLflow |
| Data Versioning | DVC |
| Model Monitoring | Evidently AI |
| API Framework | FastAPI |
| API Server | Uvicorn |
| Testing | Pytest, HTTPX |
| CI/CD | GitHub Actions |
| Containerization | Docker, Docker Compose |
| Security Scanning | Trivy |

---

## Possible Extensions

The existing pipeline can be extended with additional capabilities.

### Transformer-Based Classification

Experiment with a fine-tuned DistilBERT model using the HuggingFace Transformers library.

### Remote Data Versioning

Configure remote DVC storage using providers such as:

- Amazon S3
- Google Cloud Storage
- Azure Blob Storage

### API Metrics

Integrate Prometheus instrumentation to collect API metrics such as request count, latency, and error rates.

### Monitoring Dashboard

Connect Prometheus metrics to Grafana for monitoring application performance.

---

## Contributing

Contributions and improvements can be organized through feature branches and pull requests.

A typical workflow:

```bash
git checkout -b feature/your-feature

# Make your changes

pytest tests/ -v

git add .
git commit -m "feat: describe your change"

git push origin feature/your-feature
```

Open a pull request after reviewing your changes and confirming that the tests pass.

