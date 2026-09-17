# AGENTS.md

## Repository Overview

- Python/Gradio Stable Diffusion Web UI with AMD GPU support.
- Python 3.10.6 is used by test CI; `webui.sh` falls back from `python3.10` to `python3`.
- This is not a monorepo. There is no checked-in build command.

## Setup and Run

### Linux

Install the system packages for your distribution listed in `README.md`, then run:

```bash
./webui.sh
```

- The launcher creates `venv/`, installs Python/runtime dependencies, and starts the development server.
- Put local overrides such as `python_cmd`, `COMMANDLINE_ARGS`, or `TORCH_COMMAND` in `webui-user.sh`; do not edit `webui.sh` for local configuration.
- Prepare dependencies without keeping the server running:

```bash
python launch.py --skip-torch-cuda-test --exit
```

### Windows

```bat
webui-user.bat
```

- Configure `PYTHON`, `GIT`, `VENV_DIR`, and `COMMANDLINE_ARGS` in `webui-user.bat`.

### Test and JavaScript Dependencies

```bash
pip install wait-for-it -r requirements-test.txt
npm install
```

## Tests

Most tests call the API at `http://127.0.0.1:7860` and require the test server. Start it in one terminal:

```bash
python -m coverage run --data-file=.coverage.server launch.py --skip-prepare-environment --skip-torch-cuda-test --test-server --do-not-download-clip --no-half --disable-opt-split-attention --use-cpu all --api-server-stop
```

Then run tests in another terminal:

```bash
wait-for-it --service 127.0.0.1:7860 -t 20
python -m pytest -vv --junitxml=test/results.xml --cov . --cov-report=xml --verify-base-url test
```

Specific test examples:

```bash
# A unit-test file that does not call the API
python -m pytest -vv test/test_torch_utils.py

# One API test; requires the test server
python -m pytest -vv --verify-base-url test/test_txt2img.py::test_txt2img_simple_performed

# Tests selected by name
python -m pytest -vv -k "img2img"
```

Stop the CI-style test server:

```bash
curl -XPOST http://127.0.0.1:7860/sdapi/v1/server-stop
```

## Lint and Format

Install the Python linter version used in CI:

```bash
pip install ruff==0.3.3
```

Run before committing:

```bash
ruff check .
npm run lint
```

Apply the repository's available JavaScript auto-fixes:

```bash
npm run fix
```

- No Python formatter command is configured; Ruff is configured as a linter in `pyproject.toml`.

## Pull Requests

- Target `dev`. The `Pull requests can't target master branch` workflow intentionally fails PRs opened against `master`.
- No branch-name convention or commit-message format is defined in checked-in repository files.
- Complete the PR template: description, change summary, fixed issues, screenshots/videos when relevant, self-review, style compliance, and tests.
- PR workflows define these CI jobs:
  - `ruff`: `ruff check .`
  - `eslint`: `npm run lint`
  - `tests on CPU with empty model`: launches a CPU-only test server and runs the full pytest suite.

## Key Directories

- `modules/`: application core, launch/runtime support, APIs, model backends, processing, and UI assembly.
- `modules/api/`: HTTP API implementation and request/response models.
- `modules/dml/`: DirectML-specific backend support.
- `modules/onnx_impl/`: ONNX Runtime integration.
- `javascript/`: browser-side Web UI behavior; covered by ESLint.
- `scripts/`: built-in generation and post-processing scripts loaded by the Web UI.
- `extensions-builtin/`: extensions shipped with the repository.
- `extensions/`: user-installed extensions; excluded from Ruff checks.
- `test/`: pytest suite, fixtures, test inputs, and generated test outputs.
- `configs/`: Stable Diffusion and Olive model/runtime configuration files.
- `html/`: HTML templates and static fragments used by the UI.
- `models/`: local model storage by model type; do not commit downloaded weights.
- `repositories/`: dependency repositories cloned by the launcher.
- `embeddings/`: local textual-inversion embeddings.
- `textual_inversion_templates/`: prompt templates for textual-inversion and hypernetwork training.
- `.github/workflows/`: lint, test, and PR-target workflow definitions.
