# 🧠 CNN Image Classifier — DLOps Assignment 1

This project is an **end-to-end Deep Learning pipeline** built following **DLOps (Deep Learning Operations)** principles. It implements a CNN (Convolutional Neural Network) image classifier that is modular, scalable, and reproducible.

---

## 📌 Why Are We Building This?

In the real world, just training a model is not enough. A proper ML project needs:

- **Reproducibility** — anyone, anywhere should be able to get the same results
- **Modularity** — clean, maintainable code (research code separate from production code)
- **Version Control** — track data and models with DVC, not just code with Git
- **Automation** — CI/CD pipeline via GitHub Actions so every commit is automatically tested
- **Scalability** — if data grows or the model changes tomorrow, update just one place

**The goal is to learn how to build a proper ML system, not just a model.**

---

## 🗂️ Project Structure

```
dlops_ba_a1/
├── .github/workflows/       # CI/CD pipeline (GitHub Actions)
├── artifacts/
│   └── data_ingestion/      # Downloaded & unzipped data stored here
├── config/
│   └── config.yaml          # Data source URL, paths — all config here
├── logs/                    # Runtime logs
├── research/
│   └── data_ingestion.ipynb # Experiment/prototype notebook
├── src/cnnClassifier/
│   ├── components/          # Core logic (data_ingestion.py, etc.)
│   ├── config/              # ConfigurationManager
│   ├── constants/           # Paths to config files
│   ├── entity/              # Dataclasses (config entities)
│   ├── pipeline/            # Stage-wise pipeline files
│   └── utils/               # Helper functions (read_yaml, create_dirs)
├── templates/               # HTML templates (for web app)
├── main.py                  # Entry point — all stages run from here
├── dvc.yaml                 # DVC pipeline definition
├── params.yaml              # Model hyperparameters
├── requirements.txt         # Python dependencies
└── setup.py                 # Package install config
```

---

## ⚙️ How the Project Works — Step by Step

### Step 1: Environment Setup
```bash
uv venv
```
**Why?** Every project should have its own isolated environment so dependencies don't conflict.

---

### Step 2: Generate Project Template
```bash
python template.py
```
**Why?** This script creates all folders and empty files at once. Creating them manually every time is error-prone.

---

### Step 3: Define Constants
`src/cnnClassifier/constants/__init__.py` defines the paths to `config.yaml` and `params.yaml`.

**Why?** Hard-coding paths inside business logic is bad practice. Manage them from one place.

---

### Step 4: Write Utilities
`src/utils/common.py` contains reusable helper functions:
- `read_yaml()` — reads YAML config files
- `create_directories()` — creates folders programmatically

**Why?** These tasks are needed repeatedly — keep them in one reusable place.

---

### Step 5: Define Config
`config/config.yaml` holds:
- Data download URL
- Zip file path
- Unzip destination

**Why?** Keep configuration separate from code — if the URL changes tomorrow, update only the YAML, not the code.

---

### Step 6: Research Notebook
`research/data_ingestion.ipynb` — write and test code experimentally first.

**Why?** Notebooks let you freely try and fail. Once everything works, convert it into clean production code.

---

### Step 7: Entity (Config Dataclass)
Create `DataIngestionConfig` dataclass in `src/entity/config_entity.py`.

**Why?** Using structured config objects gives type safety and IDE autocomplete support.

---

### Step 8: Configuration Manager
Create `ConfigurationManager` class in `src/config/configuration.py` — it reads the YAML and returns a config entity.

**Why?** Single Responsibility Principle — one class handles all config loading.

---

### Step 9: Build the Component
`src/components/data_ingestion.py` contains the actual logic:
- Download data
- Extract zip
- Save to `artifacts/data_ingestion/`

**Why?** Business logic should live in its own component — easy to test and swap out.

---

### Step 10: Build the Pipeline Stage
`src/pipeline/stage_01_data_ingestion.py` — wraps the full flow of one stage into a class.

**Why?** Each stage (ingestion, training, evaluation) should be an independent, runnable unit.

---

### Step 11: Main Entry Point
All stages are called sequentially in `main.py`.

```bash
python main.py
```

After running, you'll see the downloaded and unzipped data in `artifacts/data_ingestion/`.

**Why?** The entire pipeline runs with a single command — that's the whole point of automation.

---

## 🔧 Installation

```bash
git clone https://github.com/1602saurab/dlops_ba_a1.git
cd dlops_ba_a1

uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

pip install -r requirements.txt
pip install -e .
```

---

## ▶️ Run

```bash
python main.py
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Python | Core language |
| TensorFlow/Keras | CNN model |
| DVC | Data & pipeline versioning |
| GitHub Actions | CI/CD automation |
| uv | Fast virtual environment |
| PyYAML | Config file parsing |

---

## 📊 Pipeline Stages (DVC)

```
Data Ingestion → [Model Training] → [Evaluation] → [Deployment]
```
*(This assignment covers Stage 1 — Data Ingestion)*

---

## 💡 Key Takeaway

> Writing `model.fit()` is not ML engineering.  
> Real DLOps means: **modular code + config management + versioning + automation**.