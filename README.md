# 🧠 Brain Tumor Detection — DLOps Pipeline

End-to-end TensorFlow/Keras CNN pipeline for **Brain Tumor Detection** (Tumor vs No Tumor) built with DVC versioning, a Flask web app, and CI/CD via GitHub Actions.

---

## 📌 What This Project Does

Detects the presence of a brain tumor from MRI images using a fine-tuned **VGG16** model. The pipeline is modular, reproducible, and production-ready following **DLOps** principles.

---

## 🗂️ Dataset

| Property | Details |
|---|---|
| Source | [Br35H Brain Tumor Detection](https://huggingface.co/datasets/gauravgocher712/brain-tumor-detection) |
| Classes | `yes/` (Tumor) and `no/` (No Tumor) |
| Total Images | ~253 images |
| Format | JPEG |
| Hosted On | Hugging Face |

---

## ⚙️ Model & Hyperparameters (`params.yaml`)

| Parameter | Value |
|---|---|
| IMAGE_SIZE | 64×64×3 |
| BATCH_SIZE | 32 |
| EPOCHS | 1 |
| CLASSES | 2 |
| WEIGHTS | imagenet (transfer learning) |
| LEARNING_RATE | 0.01 |
| AUGMENTATION | True |
| INCLUDE_TOP | False |

---

## 🗂️ Project Structure

```
DLOP-S/
├── .github/workflows/       # CI/CD pipeline (GitHub Actions)
├── artifacts/               # Generated outputs (not source code)
│   ├── data_ingestion/      # Downloaded & extracted dataset
│   ├── prepare_base_model/  # VGG16 base & updated model
│   ├── prepare_callbacks/   # TensorBoard logs & checkpoints
│   └── training/            # Final trained model
├── config/
│   └── config.yaml          # Central config (paths, dataset URL)
├── logs/                    # Runtime logs
├── research/                # Prototype notebooks
├── src/cnnClassifier/       # Main source package
│   ├── components/          # Core logic components
│   ├── config/              # ConfigurationManager
│   ├── constants/           # Paths to config files
│   ├── entity/              # Config dataclasses
│   ├── pipeline/            # Stage-wise pipeline files
│   └── utils/               # Helper functions
├── templates/
│   └── index.html           # Flask web UI
├── app.py                   # Flask web server
├── main.py                  # Pipeline entry point
├── dvc.yaml                 # DVC pipeline definition
├── params.yaml              # Model hyperparameters
├── requirements.txt         # Python dependencies
└── setup.py                 # Package install config
```

---

## 📄 Root Files

| File | Purpose |
|---|---|
| `app.py` | Flask web server. Routes: `/` (UI), `/train` (re-runs pipeline), `/predict` (classifies uploaded MRI image) |
| `main.py` | Master entry point — runs all 4 pipeline stages in sequence |
| `template.py` | One-time scaffolding script that creates project folder/file skeleton |
| `setup.py` | Package install config |
| `params.yaml` | Model hyperparameters read by DVC stages |
| `dvc.yaml` | DVC pipeline definition — 4 stages with deps/params/outputs |
| `requirements.txt` | All pip dependencies |
| `scores.json` | Evaluation metrics (loss, accuracy) — tracked as DVC metric |
| `library.txt` | Personal notes on Python imports used — learning reference |
| `steps.txt` | Day-by-day build log documenting incremental development |
| `.gitignore` | Git-ignored paths |

---

## ⚙️ `config/config.yaml`

Central configuration file. Controls all paths and the dataset download URL.

```yaml
data_ingestion:
  source_URL: https://huggingface.co/datasets/gauravgocher712/brain-tumor-detection/resolve/main/brain%20tumer.zip

prepare_base_model:
  base_model_path: artifacts/prepare_base_model/base_model.h5
  updated_base_model_path: artifacts/prepare_base_model/base_model_updated.h5

training:
  trained_model_path: artifacts/training/model.h5
```

---

## 🧩 `src/cnnClassifier/` — Main Package

### `components/`

| File | Purpose |
|---|---|
| `data_ingestion.py` | Downloads dataset zip from Hugging Face and extracts to `artifacts/data_ingestion/` |
| `prepare_base_model.py` | Loads pretrained VGG16, freezes layers, attaches softmax classification head, saves model |
| `prepare_callbacks.py` | Builds Keras `TensorBoard` and `ModelCheckpoint` callbacks |
| `training.py` | Sets up train/validation `ImageDataGenerator` with augmentation, fits and saves the model |
| `evaluation.py` | Evaluates trained model on validation split, writes loss/accuracy to `scores.json` |

### `pipeline/`

| File | Purpose |
|---|---|
| `stage_01_data_ingestion.py` | Pipeline wrapper for data download and extraction |
| `stage_02_prepare_base_model.py` | Pipeline wrapper for VGG16 base model preparation |
| `stage_03_training.py` | Pipeline wrapper for model training |
| `stage_04_evaluation.py` | Pipeline wrapper for model evaluation |
| `predict.py` | `PredictionPipeline` — loads trained model, classifies single MRI image as `Tumor Detected` or `No Tumor` |

### `config/`

| File | Purpose |
|---|---|
| `configuration.py` | `ConfigurationManager` — reads `config.yaml` and `params.yaml`, builds typed config objects for each stage |

### `entity/`

| File | Purpose |
|---|---|
| `config_entity.py` | Frozen dataclasses for each stage config: `DataIngestionConfig`, `PrepareBaseModelConfig`, `PrepareCallbacksConfig`, `TrainingConfig`, `EvaluationConfig` |

### `utils/`

| File | Purpose |
|---|---|
| `common.py` | Shared helpers: `read_yaml`, `create_directories`, `save_json`, `load_json`, base64 encode/decode for images |

### `constants/`

| File | Purpose |
|---|---|
| `__init__.py` | Defines fixed paths to `config.yaml` and `params.yaml` |

---

## 🔬 `research/` — Prototype Notebooks

| File | Purpose |
|---|---|
| `data_ingestion.ipynb` | Prototype of data download/extraction before converting to component |
| `prepare_base_model.ipynb` | Prototype of VGG16 setup before converting to component |
| `03_prepare_callbacks.ipynb` | Prototype of TensorBoard/checkpoint callback setup |
| `training.ipynb` | Prototype of training loop before converting to component |
| `evaluation.ipynb` | Prototype of evaluation logic before converting to component |
| `trials.ipynb` | General scratchpad/experiments |

---

## 🌐 `templates/`

| File | Purpose |
|---|---|
| `index.html` | Brain Tumor Detection web UI — upload MRI image, displays `⚠️ Tumor Detected` or `✅ No Tumor` |

---

## 🔄 DVC Pipeline (`dvc.yaml`)

```
data_ingestion → prepare_base_model → training → evaluation
```

| Stage | Produces |
|---|---|
| `data_ingestion` | `artifacts/data_ingestion/brain tumer/` |
| `prepare_base_model` | `artifacts/prepare_base_model/` |
| `training` | `artifacts/training/model.h5` |
| `evaluation` | `scores.json` |

Run with:
```bash
dvc repro        # runs only changed stages
python main.py   # runs all stages
```

---

## 🚀 CI/CD (`.github/workflows/main.yaml`)

| Job | Purpose |
|---|---|
| `integration` | Lint/test checks on every push to `main` |
| `build-and-push-ecr-image` | Builds Docker image, pushes to AWS ECR |
| `Continuous-Deployment` | Pulls latest ECR image, restarts container on port `8080` |

**Required GitHub Secrets:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `ECR_REPOSITORY_NAME`

---

## 🔧 Installation & Run

```bash
git clone https://github.com/gaurav712-gujjar/DLOP-S.git
cd DLOP-S

uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

pip install -r requirements.txt
pip install -e .

# Run full pipeline
python main.py

# Start web app
python app.py
```

Open `http://localhost:8080` in your browser.

---

## 📊 Results

| Metric | Value |
|---|---|
| Loss | 0.608 |
| Accuracy | 68.4% |

> Trained on CPU with `64×64` image size and 1 epoch. Use GPU with `224×224` and 10+ epochs for 90%+ accuracy.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Python | Core language |
| TensorFlow/Keras | CNN model (VGG16 transfer learning) |
| DVC | Data & pipeline versioning |
| Flask | Web application |
| GitHub Actions | CI/CD automation |
| Hugging Face | Dataset hosting |
| uv | Fast virtual environment |
| PyYAML | Config file parsing |

---

## 💡 Key Takeaway

> Writing `model.fit()` is not ML engineering.
> Real DLOps means: **modular code + config management + versioning + automation**.
