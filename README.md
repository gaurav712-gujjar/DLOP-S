# 🧠 CNN Image Classifier — DLOps Pipeline

End-to-end TensorFlow/Keras CNN pipeline (chicken fecal image classifier: Healthy vs Coccidiosis) built with DVC versioning, a Flask web app, and CI/CD via GitHub Actions.

## Root files

| File | Purpose |
|---|---|
| `app.py` | Flask web server. Routes: `/` (UI), `/train` (re-runs `main.py` via OS call), `/predict` (decodes base64 image, runs `PredictionPipeline`). |
| `main.py` | Master entry point — runs all 4 pipeline stages (ingestion → base model → training → evaluation) in sequence with logging/error handling. |
| `template.py` | One-time scaffolding script that creates the project's folder/file skeleton (used at project setup, not in normal runs). |
| `setup.py` | Package install config (currently empty/placeholder). |
| `dockerfile` | Builds a container image: Python 3.9-slim, installs `awscli` + `requirements.txt`, runs `app.py`. |
| `requirements.txt` | Pip dependencies (tensorflow, dvc, Flask, Flask-Cors, python-box, pyyaml, joblib, etc.). |
| `pyproject.toml` | `uv`/PEP 621 project metadata (separate, minimal dependency list — `tensorboard`). |
| `uv.lock` | Locked dependency versions for `uv` package manager. |
| `.python-version` | Pins Python version (`3.11`) for `uv venv`. |
| `params.yaml` | Model hyperparameters: image size, batch size, epochs, learning rate, augmentation flag, etc. Read by DVC stages. |
| `dvc.yaml` | DVC pipeline definition — 4 stages (data_ingestion, prepare_base_model, training, evaluation) with deps/params/outs for reproducible runs. |
| `dvc.lock` | DVC's lock file recording exact hashes of deps/outputs from the last pipeline run. |
| `.dvcignore` | Patterns DVC should ignore (like `.gitignore` for DVC). |
| `.dvc/config`, `.dvc/.gitignore` | DVC internal config and ignore rules. |
| `library.txt` | Personal notes explaining basic Python imports (`os`, `pathlib`, `logging`) used in the project — learning reference, not code. |
| `steps.txt` | Day-by-day build log/journal documenting how the project was developed incrementally. |
| `.gitignore` | Git-ignored paths (envs, caches, etc.). |
| `inputImage.jpg` | Sample/last image submitted to `/predict` for inference. |
| `ouptut_result.json` | Sample prediction output saved from a manual test run. |
| `scores.json` | Evaluation metrics (loss, accuracy) written by the evaluation stage — tracked as a DVC metric. |
| `logs/running_logs.log` | Runtime log file written by the project-wide logger. |

## `.github/workflows/`

| File | Purpose |
|---|---|
| `main.yaml` | CI/CD pipeline: on push to `main` runs lint/test stub jobs, then a (partially defined) "build-and-push-ecr-image" job for deployment. |
| `.gitkeep` | Empty placeholder to keep the otherwise-empty folder tracked by Git. |

## `config/`

| File | Purpose |
|---|---|
| `config.yaml` | Central config: artifact paths, dataset source URL, model/checkpoint paths for every pipeline stage. This is the **active** config read by `ConfigurationManager`. |

## `src/cnnClassifier/` — main package

| File | Purpose |
|---|---|
| `__init__.py` | Sets up the project-wide logger (writes to `logs/running_logs.log` and stdout). |
| `constants/__init__.py` | Defines fixed paths to `config.yaml` and `params.yaml`. |
| `utils/common.py` | Shared helpers: `read_yaml`, `create_directories`, `save_json`/`load_json`, `save_bin`/`load_bin`, `get_size`, and base64 encode/decode for images (used by Flask app). |
| `entity/config_entity.py` | Frozen dataclasses defining the config schema for each stage: `DataIngestionConfig`, `PrepareBaseModelConfig`, `PrepareCallbacksConfig`, `TrainingConfig`, `EvaluationConfig`. |
| `config/configuration.py` | `ConfigurationManager` class — reads `config.yaml`/`params.yaml` and builds the typed config-entity object for each stage. |
| `config/config.yaml` | **Stale duplicate** of the root `config/config.yaml` (missing the `prepare_callbacks` section) — not actually read by the app; the constants point to the root-level file. |
| `pipeline/configuration.py` | Empty placeholder file. |
| `pipeline/stage_01_data_ingestion.py` | Pipeline wrapper: downloads dataset zip and extracts it via the `DataIngestion` component. |
| `pipeline/stage_02_prepare_base_model.py` | Pipeline wrapper: builds VGG16 base model and adapts it (adds classification head) via `PrepareBaseModel`. |
| `pipeline/stage_03_training.py` | Pipeline wrapper: sets up TensorBoard/checkpoint callbacks and trains the model via `Training`. |
| `pipeline/stage_04_evaluation.py` | Pipeline wrapper: loads the trained model, evaluates on validation data, saves metrics via `Evaluation`. |
| `pipeline/predict.py` | `PredictionPipeline` — loads the trained model and classifies a single input image as `Healthy` or `Coccidiosis`. Used by Flask's `/predict` route. |
| `components/data_ingestion.py` | `DataIngestion` — downloads the dataset zip (if not present) and extracts it to `artifacts/data_ingestion/`. |
| `components/prepare_base_model.py` | `PrepareBaseModel` — loads pretrained VGG16, freezes layers, attaches a dense softmax classification head, compiles and saves it. |
| `components/prepare_callbacks.py` | `PrepareCallback` — builds Keras `TensorBoard` and `ModelCheckpoint` callbacks for training. |
| `components/training.py` | `Training` — sets up train/validation `ImageDataGenerator`s (with optional augmentation), compiles, fits, and saves the final model. |
| `components/evaluation.py` | `Evaluation` — runs the trained model against a validation split and writes loss/accuracy to `scores.json`. |

## `research/` — prototyping notebooks

| File | Purpose |
|---|---|
| `trials.ipynb` | General scratchpad/experiments notebook. |
| `data_ingestion.ipynb` | Prototype of the data download/extraction logic before it was converted into `data_ingestion.py`. |
| `prepare_base_model.ipynb` | Prototype of VGG16 base-model setup before conversion to `prepare_base_model.py`. |
| `03_prepare_callbacks.ipynb` | Prototype of TensorBoard/checkpoint callback setup. |
| `training.ipynb` | Prototype of the training loop before conversion to `training.py`. |
| `evaluation.ipynb` | Prototype of the model evaluation logic before conversion to `evaluation.py`. |

## `templates/`

| File | Purpose |
|---|---|
| `index.html` | Front-end page served at `/` — image upload UI that calls the Flask `/predict` (and `/train`) endpoints. |

## `artifacts/` — generated outputs (not source code)

| Path | Purpose |
|---|---|
| `data_ingestion/data.zip` | Downloaded raw dataset archive. |
| `data_ingestion/Chicken-fecal-images/Coccidiosis/*.jpg`, `.../Healthy/*.jpg` | Extracted, labeled training images (two classes). |
| `prepare_callbacks/checkpoint_dir/model.h5` | Best-checkpointed model saved during training. |
| `prepare_callbacks/tensorboard_log_dir/.../events.out.tfevents...` | TensorBoard logs from multiple training runs. |
| `training/.gitignore` | Excludes the large trained model file (`model.h5`) from Git. |
| `.gitignore` (top-level in `artifacts/`) | Excludes generated artifacts from Git (versioned by DVC instead). |

## Notes

- The pipeline currently has 4 stages implemented (ingestion, base model prep, training, evaluation); `dvc.yaml` defines all 4 as reproducible DVC stages.
- `params.yaml` controls hyperparameters without touching code; `config/config.yaml` controls paths/URLs the same way.
- `src/cnnClassifier/config/config.yaml` is a leftover/duplicate and can likely be deleted — the live config lives at the repo-root `config/config.yaml`.
