# Contributing

## Prerequisites

To contribute to this project, you need the following:

### Python

- [UV](https://docs.astral.sh/uv/getting-started/installation/)
- [pre-commit](https://pre-commit.com/) (via `uv tool install pre-commit`)
- [ruff](https://astral.sh/ruff) (via `uv tool install ruff`)

### Go

- [Go 1.22](https://go.dev/doc/install)
- [Mage](https://magefile.org/) (via `go install github.com/magefile/mage@v1.15.0`)
- [protoc-gen-go](https://pkg.go.dev/github.com/golang/protobuf/protoc-gen-go) (via `go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.33.0`)
- [Mockery](https://vektra.github.io/mockery/latest/) (via `go install github.com/vektra/mockery/v2@v2.43.2`)
- [Golangci-lint](https://golangci-lint.run/) (via `go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.59.1`)

### Protocol Buffer Compiler

See [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/)

Alternatively, you can use the [development container](https://containers.dev/) that includes all the required tools.

## Installation

to configure all the development environment just run `mage` target:

```bash
mage configure
```

it will configure MLflow and all the Python dependencies required by the project or run each step manually:

```bash
# Install our Python package and its dependencies
pip install -e .

# Install the dreaded psycho
pip install psycopg2-binary

# Archive the MLflow pre-built UI
tar -C /usr/local/python/current/lib/python3.8/site-packages/mlflow -czvf ./ui.tgz ./server/js/build

# Clone the MLflow repo
git clone https://github.com/jgiannuzzi/mlflow.git -b master .mlflow.repo

# Add the UI back to it
tar -C .mlflow.repo/mlflow -xzvf ./ui.tgz

# Install it in editable mode
pip install -e .mlflow.repo
```

## Run Go MLflow server

To start the mlflow-go dev server connecting to postgres just run next `mage` target:

```bash
mage dev
```

The postgres database should already be running prior to this command. By default service uses next connection string:

```
postgresql://postgres:postgres@localhost:5432/postgres
```

but it could be configured in [mage](./magefiles/run.go)

## Porting an Endpoint

If you wish to contribute to the porting of an existing Python endpoint, you can read our [dedicated guide](./docs/porting-a-new-endpoint.md).

## Run tests

The Python integration tests have been adapted to also run against the Go implementation.
Next `mage` targets are available to run different types of tests:

```bash
# Run all the available tests
mage test:all
```

```bash
# Run just MLflow Python tests
mage test:python
```

```bash
# Run specific MLflow Python tests (matches all tests containing the argument)
mage test:pythonSpecific <test_file::test_name>

#Example
mage test:pythonSpecific ".mlflow.repo/tests/tracking/test_rest_tracking.py::test_rename_experiment"
```

```bash
# Run just unit tests 
mage test:unit
```

Additionally, there is always an option to run, specific test\tests if it is necessary:

```bash
pytest tests/tracking/test_rest_tracking.py
```

To run only the tests targeting the Go implementation, you can use the `-k` flag:

```bash
pytest tests/tracking/test_rest_tracking.py -k '[go-'
```

If you'd like to run a specific test and see its output 'live', you can use the `-s` flag:

```bash
pytest -s "tests/tracking/test_rest_tracking.py::test_create_experiment_validation[go-postgresql]"
```

See the [pytest documentation](https://docs.pytest.org/en/8.2.x/how-to/usage.html#specifying-which-tests-to-run) for more details.

```bash
# Build the Go binary in a temporary directory
libpath=$(mktemp -d)
python -m mlflow_go.lib . $libpath

# Run the tests (currently just the server ones)
MLFLOW_GO_LIBRARY_PATH=$libpath pytest --confcutdir=. \
  .mlflow.repo/tests/tracking/test_rest_tracking.py \
  .mlflow.repo/tests/tracking/test_model_registry.py \
  .mlflow.repo/tests/store/tracking/test_sqlalchemy_store.py \
  .mlflow.repo/tests/store/model_registry/test_sqlalchemy_store.py \
  -k 'not [file'

# Remove the Go binary
rm -rf $libpath

# If you want to run a specific test with more verbosity
# -s for live output
# --log-level=debug for more verbosity (passed down to the Go server/stores)
MLFLOW_GO_LIBRARY_PATH=$libpath pytest --confcutdir=. \
  .mlflow.repo/tests/tracking/test_rest_tracking.py::test_create_experiment_validation \
  -k 'not [file' \
  -s --log-level=debug
```