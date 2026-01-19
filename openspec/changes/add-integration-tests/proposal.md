# Change: Add Integration Tests for MLOps Pipeline

## Why
The project currently only has unit tests for the data processor, but lacks integration tests that validate end-to-end workflows across the MLOps pipeline. Integration tests are essential to ensure components work together correctly in realistic scenarios involving Databricks, MLflow, and Unity Catalog interactions.

## What Changes
- Add integration test suite covering end-to-end data pipeline workflows (data loading, preprocessing, Unity Catalog writes)
- Add integration tests for model training and MLflow experiment tracking workflows
- Add integration tests for model registration and versioning in Unity Catalog
- Add integration tests for model serving endpoint deployment and inference
- Add integration tests for monitoring table creation and data flow
- Add pytest markers to distinguish integration tests from unit tests (`@pytest.mark.integration`)
- Add configuration for running integration tests with Databricks Connect
- Leverage existing `generate_test_data(num_rows=20)` function for fast test execution with small datasets
- Create reusable test fixtures in `conftest.py` providing small (20-100 row) datasets
- Ensure integration tests complete in 10-15 minutes using scaled-down data
- Update test documentation with instructions for running integration vs unit tests

## Impact
- Affected specs: New capability `testing` (no existing spec)
- Affected code:
  - `tests/` - New integration test files under `tests/integration/`
  - `pyproject.toml` - New pytest markers for integration tests
  - `conftest.py` - New fixtures for integration test setup (Databricks connections, test catalogs)
  - `README.md` - Documentation updates for running tests
