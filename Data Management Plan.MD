# 📄 Data Management Plan (DMP)

## Project: MAPLN Artemis RAG Assistant

***

# 1. Project Overview

This project develops a **Retrieval-Augmented Generation (RAG) assistant** using data provided through a Kaggle competition. The system leverages document retrieval and large language models to generate context-aware responses.

The DMP defines how data is acquired, processed, versioned, validated, and tracked throughout the lifecycle of the project to ensure **reproducibility, traceability, and quality control**.

***

# 2. Data Source

* **Dataset**: MAPLN Artemis – RAG Assistant (Kaggle Competition)
* **Access**: Restricted to Kaggle participants
* **Download date**: June 2026
* **License**: Governed by Kaggle competition terms
* **Acquisition method**: Kaggle API (`kaggle competitions download`)

### Versioning Strategy

* Raw dataset treated as an **immutable snapshot**
* Managed using **DVC (Data Version Control)** to ensure reproducibility

***

# 3. Data Description

### Data Types

* Knowledge base documents (text corpus)
* Query inputs
* Ground truth answers (for evaluation)

### Data Formats

* JSON (primary)
* CSV (evaluation datasets)
* TXT (optional raw documents)

### Estimated Size

* \~1–5 GB (typical for RAG competition datasets)

### Structure

```bash
data/
├── raw/        # original Kaggle data (immutable)
├── interim/    # cleaned / parsed documents
└── processed/  # embeddings, chunks, indices
```

***

# 4. Storage & Infrastructure

### Storage Locations

* **Local environment**: development and experimentation
* **DVC remote**:
  * Option A: Google Drive
  * Option B: AWS S3 (preferred for production-like setup)

### Data Organization

* Raw data is read-only
* All transformations produce new versioned outputs

### Backup Strategy

* DVC ensures versioned snapshots
* Remote storage provides redundancy

***

# 5. Data Versioning (DVC)

### Objectives

* Track large datasets independently of Git
* Ensure full reproducibility of pipelines
* Enable rollback to any data version

### Pipeline Definition (`dvc.yaml`)

```yaml
stages:
  preprocess:
    cmd: python src/preprocess.py
    deps:
      - data/raw/
    outs:
      - data/interim/

  chunk:
    cmd: python src/chunk.py
    deps:
      - data/interim/
    outs:
      - data/processed/chunks/

  embed:
    cmd: python src/embed.py
    deps:
      - data/processed/chunks/
    outs:
      - data/processed/embeddings/

  index:
    cmd: python src/index.py
    deps:
      - data/processed/embeddings/
    outs:
      - data/processed/index/
```

### Key Guarantees

* Every artifact is reproducible
* Data lineage is explicit
* Data + pipeline = deterministic execution

***

# 6. Experiment Tracking (MLflow)

### Purpose

Track experimentation across retrieval and generation to systematically optimize performance.

### Tracked Components

#### Parameters

* Chunk size (e.g., 256, 512 tokens)
* Embedding model (e.g., SentenceTransformers)
* Top-K retrieval
* Prompt template
* LLM type

#### Metrics

* Retrieval:
  * Recall\@K
  * Precision\@K
* Generation:
  * BLEU / ROUGE
  * Semantic similarity
  * Custom evaluation score

#### Artifacts

* Trained indices (FAISS / vector store)
* Model checkpoints
* Predictions
* Evaluation reports

### Integration with Pipeline

* Each MLflow run is tied to:
  * Git commit
  * DVC data version
  * Pipeline configuration

👉 This creates **full experiment reproducibility**

***

# 7. Data Processing Pipeline

### Pipeline Steps

#### Step 1: Ingestion

* Download dataset from Kaggle
* Store under `data/raw/`

#### Step 2: Preprocessing

* Normalize text (lowercase, remove noise)
* Remove invalid or empty entries

#### Step 3: Chunking

* Split documents into manageable pieces
* Maintain overlap to preserve context

#### Step 4: Embedding Generation

* Generate vector embeddings for chunks
* Store as serialized arrays

#### Step 5: Indexing

* Build retrieval index (FAISS or similar)

***

# 8. Data Quality & Validation

### Implemented Checks

* Missing or null entries
* Duplicate documents or queries
* Text encoding errors
* Empty or excessively short chunks

### Optional Enhancements

* Schema validation (JSON schema / Pydantic)
* Logging anomalies during processing
* Unit tests for pipeline stages

***

# 9. Security & Access Control

### Data Sensitivity

* Moderate (competition dataset, not public redistributable)

### Access Control

* Local developer environment
* Kaggle competition restrictions enforced

### Policies

* No redistribution of raw data
* Only derived outputs shared when allowed

***

# 10. Data Sharing & Use

### What is Shared

* Code (GitHub repository)
* DVC pipeline definitions
* MLflow experiment results
* Trained models (if permitted)

### What is NOT Shared

* Raw Kaggle dataset (license restricted)

***

# 11. Reproducibility Strategy

### Full Reproducibility Stack

| Component   | Tool   |
| ----------- | ------ |
| Code        | Git    |
| Data        | DVC    |
| Experiments | MLflow |

### Reproduce Pipeline

```bash
git clone <repo>
dvc pull
dvc repro
mlflow ui
```

### Guarantees

* Exact rerun of experiments
* Traceable results
* Deterministic pipelines

***

# 12. Risks & Limitations

* Dataset may contain noisy labels
* Retrieval performance depends heavily on chunking strategy
* Embedding model choice significantly affects results
* Evaluation metrics for RAG may not fully capture semantic correctness

***

# 13. Roles & Responsibilities

| Role            | Responsibility                           |
| --------------- | ---------------------------------------- |
| Developer (You) | Data pipeline, modeling, evaluation      |
| System Owner    | Versioning, experiments, reproducibility |
| Reviewer        | Validation and interpretation of results |

***

# 14. Future Improvements

* Integrate vector database (e.g., Pinecone, Weaviate)
* Add API layer (FastAPI) for serving responses
* Implement automated evaluation pipelines
* Add data drift monitoring for evolving datasets

***
