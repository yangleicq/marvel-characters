# Project Context

## Purpose
This is an end-to-end MLOps project that demonstrates machine learning lifecycle management using Databricks. The project builds classification models to predict Marvel character survival status (Alive/Dead) based on their attributes (physical characteristics, universe, origin, powers, etc.). It showcases best practices for data preprocessing, model training, experiment tracking, model deployment, and monitoring in a production MLOps environment.

## Tech Stack
- **Python**: 3.12 (managed with UV package manager)
- **Platform**: Databricks (Serverless v3, Unity Catalog)
- **ML Frameworks**: scikit-learn 1.7.0, LightGBM 4.6.0
- **Data Processing**: PySpark 3.4+, pandas 2.3.0, numpy 1.26.4
- **ML Operations**: MLflow 3.1.1 (experiment tracking, model registry)
- **Configuration**: Pydantic 2.11.7, PyYAML
- **Deployment**: Databricks Asset Bundles (DABs)
- **Testing**: pytest 8.0+, pytest-cov 5.0+
- **Code Quality**: Ruff (linting and formatting), pre-commit 4.1+
- **SDK**: databricks-sdk 0.55.0

## Project Conventions

### Code Style
- **Formatter/Linter**: Ruff with strict configuration
- **Line Length**: 120 characters maximum
- **Type Hints**: Required on all functions (flake8-annotations enabled)
- **Docstrings**: Required for all public modules, classes, and functions (pydocstyle rules)
  - Use imperative mood ("Process data" not "Processes data")
  - Include parameter descriptions with `:param` and return types with `:return:`
- **Naming Conventions**:
  - snake_case for functions, variables, modules
  - PascalCase for classes
  - UPPER_CASE for constants
  - Descriptive names (avoid abbreviations)
- **Import Organization**: Sorted with isort (standard library → third-party → local)
- **Code Simplicity**: Prefer clear, simple solutions over complex abstractions (flake8-simplify enabled)

### Architecture Patterns
- **Modular Structure**: 
  - `src/marvel_characters/` contains all application code
  - Separate modules: `data_processor.py`, `models/`, `monitoring.py`, `serving/`
  - Configuration centralized in `config.py` using Pydantic models
- **Configuration Management**:
  - Environment-specific configs (dev, acc, prd) in YAML files
  - Pydantic models validate and type-check configurations
  - Catalog/schema names differ per environment
- **Data Pipeline**:
  - Load raw data → preprocess → split → save to Unity Catalog
  - Enable Change Data Feed for incremental processing
  - Timestamp all data writes for audit trails
- **Model Management**:
  - MLflow for experiment tracking and model registry
  - Custom model wrappers for complex inference logic
  - Version tagging with git SHA and branch info
- **Deployment**:
  - Databricks Asset Bundles for infrastructure-as-code
  - Separate targets (dev/acc/prd) with distinct workspace paths
  - Model serving endpoints for real-time inference

### Testing Strategy
- **Framework**: pytest with coverage reporting
- **Coverage Target**: Track with `pytest-cov`, report to XML for CI/CD
- **Test Organization**:
  - Tests mirror `src/` structure under `tests/`
  - Use `conftest.py` for shared fixtures
  - Mark tests excluded from CI with `@pytest.mark.ci_exclude`
- **Test Scope**:
  - Unit tests for data processors, models, utilities
  - Integration tests may use Databricks Connect (optional dev dependency)
- **Running Tests**: `uv run pytest` (automatically activates environment)

### Git Workflow
- **Branching**: Feature branches merged to `main`
- **Commit Tags**: Git SHA and branch tracked in MLflow runs and model metadata
- **Environment Variables**:
  - `git_sha`: Current commit hash
  - `branch`: Current branch name
  - Used for reproducibility and audit trails

## Domain Context
- **Dataset**: Marvel Characters Dataset from Kaggle
  - Features: Height, Weight, Universe, Identity, Gender, Marital Status, Teams, Origin, Magic, Mutant
  - Target: Binary classification (Alive/Dead)
  - ~600+ characters after preprocessing
- **Feature Engineering**:
  - Normalize universes (group rare ones as "Other")
  - Binary flags for team membership, magic, mutant status
  - Categorical encoding for origin types (Human, Mutant, Asgardian, Alien, etc.)
  - Handle missing values with domain knowledge (e.g., "Unknown" for identity)
- **Model Types**:
  - Basic models: Standard scikit-learn/LightGBM classifiers
  - Custom models: MLflow pyfunc wrappers with preprocessing pipelines
- **Monitoring**:
  - Track prediction drift and data quality
  - Lakehouse monitoring tables for dashboards
  - Synthetic data generation for testing drift scenarios

## Important Constraints
- **Python Version**: Must use Python >=3.12, <3.13 (strict runtime requirement)
- **Databricks Environment**: Serverless compute version 3
- **Unity Catalog**: All data must be saved to catalog.schema.table format
  - Dev: `mlops_dev.marvel_characters.*`
  - Acc: `mlops_acc.marvel_characters.*`
  - Prd: `mlops_prd.marvel_characters.*`
- **Workspace Profile**: Uses `marvelous` profile for authentication
- **Deployment Constraints**:
  - Wheel packages built with `uv build` 
  - Bundle artifacts deployed via Databricks CLI
  - Schedules default to PAUSED in non-production
- **Code Quality Gates**: All Ruff rules must pass before deployment

## External Dependencies
- **Databricks Workspace**: `https://dbc-7a05be8a-5448.cloud.databricks.com/`
- **Kaggle Dataset**: Marvel Characters Dataset (loaded from `data/marvel_characters_dataset.csv`)
- **MLflow Tracking Server**: Managed by Databricks
- **Model Registry**: Databricks Unity Catalog (UC) model registry
- **Model Serving**: Databricks Model Serving endpoints
- **Artifact Storage**: Databricks DBFS and Unity Catalog volumes
- **Monitoring**: Databricks Lakehouse Monitoring (bundle_monitoring.yml)
- **CI/CD**: GitHub integration for status updates (`scripts/post_commit_status.py`)
