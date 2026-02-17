# Running tests for this project

## Run tests locally

Install dependencies:

```bash
pip install -r requirements.txt
```

Run the test suite:

```bash
python -m pytest -v
```

## Build and run tests with Docker

Build the image:

```bash
docker build -t ai-experts-assignment-3 .
```

Run the tests in the container:

```bash
docker run --rm ai-experts-assignment-3
```
