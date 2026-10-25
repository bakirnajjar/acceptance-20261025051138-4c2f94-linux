# Synthetic pricing sample (Python)

Synthetic sample for Futuristic Modernization acceptance testing. It contains no real
business data, credentials or deployment configuration.

Use Python 3.12 or newer. In an environment with `uv` and Python 3.12 at
`/usr/bin/python3.12` (where a bare `python` command may not be installed),
compile the source and run all tests with the dependencies in `requirements.txt`:

```shell
python3.12 -m compileall -q pricing.py test_pricing.py
uv run --no-project --python /usr/bin/python3.12 --with-requirements requirements.txt python -m pytest
```

`uv` provides the `python` command inside its isolated run environment; no
system-wide pytest installation is required.
