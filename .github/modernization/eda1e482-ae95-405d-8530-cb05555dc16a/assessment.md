# Phase 1 assessment — eda1e482-ae95-405d-8530-cb05555dc16a

## Summary

**Observed:** This six-file synthetic pricing sample declares Python `>=3.10` and runs on the installed Python 3.12.3 without source changes. The Python 3.12 compilation gate passed; all six existing tests passed. No lint gate is configured. No services are needed. The final validation/report must include marker `acceptance-20261025051138-4c2f94-linux-validated`; this assessment does **not** claim final validation.

**Recommendation:** In-place modernization to an explicit Python 3.12 minimum, after one behavior-preserving characterization/regression test. Overall risk **low** for code compatibility, with a **medium** reproducibility risk from an unbounded test dependency and absent `python` executable in this environment.

## Inventory / Stack inventory

`git ls-files | wc -l` returned 6: two `.py`, one `.toml`, one `.txt`, one `.md`, and one `.gitignore`. No solution, other module, package lock, configured CI workflow, container/deployment manifest, or executable build wrapper is tracked (`git ls-files`; repository inspection, 2026-10-25).

| Unit | Path | Kind | Stack / current version evidence | Candidate target |
|---|---|---|---|---|
| Pricing | `pricing.py` | Single-file library | Python, comment-style annotations (`pricing.py:3-5,11-12,25-26`) | Python 3.12, preserve integer-cent arithmetic |
| Package metadata | `pyproject.toml` | Project configuration | `requires-python = ">=3.10"` (`pyproject.toml:1-8`) | Declare `>=3.12` after compatibility check |
| Test suite | `test_pricing.py` | Six pytest tests | pytest import (`test_pricing.py:1`); `pytest>=7` (`requirements.txt:1`) | Test under Python 3.12 with resolved pytest |
| Usage instructions | `README.md` | Documentation | Says written for Python 3.10, calls `python -m pip install -r requirements.txt` and `python -m pytest` (`README.md:6-11`) | Document working 3.12 setup and gates |

Project graph: `test_pricing.py:3` imports `pricing.py`; no other local imports. Configuration keys: `project.name`, `project.version`, `project.description`, `project.requires-python`, `tool.pytest.ini_options.testpaths` (`pyproject.toml:1-8`). No secrets or external-service configuration found.

## Version and support facts / Support status

Checked 2026-10-25 against official vendor documentation; a declared lower bound is **not** evidence of the interpreter previously used in deployment.

| Fact | Value | Source URL | Checked on |
|---|---|---|---|
| Declared Python minimum | 3.10 (`pyproject.toml:5`); README says developed for 3.10 (`README.md:6`) | Repository paths above | 2026-10-25 |
| Python 3.10 branch | End of life 2026-10-01; lower bound now allows an unsupported interpreter | https://devguide.python.org/versions/ | 2026-10-25 |
| Python 3.12 target | Security-maintenance branch, scheduled end of life October 2028; locally installed interpreter 3.12.3 (`python3 --version`) | https://devguide.python.org/versions/ | 2026-10-25 |
| pytest compatibility | pytest 9.0+ minimum Python 3.10; resolved pytest 9.1.1 executed on 3.12.3 | https://docs.pytest.org/en/stable/backwards-compatibility.html#python-version-support and baseline test output | 2026-10-25 |
| pytest support end date | Unverified: no version-specific end-of-support date established | https://docs.pytest.org/en/stable/backwards-compatibility.html | 2026-10-25 |

Python 3.12 is the requested target, not the newest Python branch. There is no framework or runtime image declared beyond Python.

## Dependencies and vulnerabilities / Dependencies

`requirements.txt:1` specifies **only** the unpinned direct test dependency `pytest>=7`; production module uses the standard library `typing` (`pricing.py:3`). No lockfile or private index configuration is tracked. This baseline resolved pytest 9.1.1 and transitive `pluggy` 1.7.0, `iniconfig` 2.3.1, `pygments` 2.21.0 (`uvx --from pip-audit pip-audit -r requirements.txt --format json`, exit 0). The resolver may choose different versions later.

| Package | Current specification / baseline resolution | Supports target? | Deprecated or unmaintained? | Known vulnerabilities | Action |
|---|---|---|---|---|---|
| pytest | `>=7` / 9.1.1 | Yes; vendor compatibility table above, confirmed by tests | No indication of deprecation in vendor documentation; full maintenance status unverified | None found for resolved 9.1.1 | Consider bounded or locked development dependency in a later scoped task |
| pluggy, iniconfig, pygments | Transitive; 1.7.0, 2.3.1, 2.21.0 | Resolved and executed on 3.12; independent vendor support window unverified | Unverified | None found for the resolved versions | Audit again after any dependency change |

Audit command completed in 20.843 s with exit 0 and reported **0 known vulnerabilities** (critical 0, high 0, medium 0, low 0); this is a point-in-time result for resolved dependencies, not a guarantee for every version allowed by `>=7`. Raw audit JSON/log retained only in ignored `.modernization-scratch/baseline/`. No advisory or fixed version to report.

## Build, test and CI / Test baseline

Environment: Linux; `/usr/bin/python3.12` = Python 3.12.3, `python3 -m pip --version` = pip 24.0, uv 0.12.23. Bare `python` is **not installed** and system Python 3.12 does not have pytest (`python3 -m pytest --version` reports missing module). The README commands (`README.md:8-11`) therefore require environment setup; the test used an isolated uv environment resolving the same `requirements.txt`, without changing repository dependencies. Both gates used `TZ=UTC`, `LANG=C.UTF-8`, `PYTHONPYCACHEPREFIX=.modernization-scratch/pycache`; the test also used `UV_CACHE_DIR=.modernization-scratch/uv-cache`. Timings are wall-clock.

| Command | Source | Exit | Duration | Result |
|---|---|---:|---:|---|
| `python3.12 -m compileall -q pricing.py test_pricing.py` | Repository files; explicit real syntax/build gate (no distribution build configured) | 0 | 0.413 s | GREEN; no output or warnings |
| `uv run --no-project --python /usr/bin/python3.12 --with-requirements requirements.txt python -m pytest -rA --junitxml=.modernization-scratch/baseline/pytest.xml` | Full README test suite adapted to available interpreter and isolated requirements | 0 | 2.741 s | GREEN; **6 passed / 0 failed / 0 skipped**, 0 warnings reported |

Counts come from six `<testcase>` nodes in `.modernization-scratch/baseline/pytest.xml` (six passed, zero failure/error/skipped), not only console output. No failed test IDs or first error lines; no rerun required because baseline is green. Raw logs and JUnit XML are ignored local scratch outputs and are not committed. A package build (`python -m build`) is **not configured**; compilation is the defensible real build gate for the current single-file module. Lint is not configured. No tracked `.github/workflows`, Azure Pipelines, Jenkinsfile, or CI runtime pins exist.

**Recommended later gates:** retain the exact full Python 3.12 compile command and full pytest command above (or an equivalent provisioned Python 3.12 environment running `python -m pytest -rA --junitxml=...`), compare test discovery/counts with this baseline, and audit resolved dependencies. Do not substitute a subset of tests. No lint command can be prescribed from existing project configuration.

## Blockers and hotspots

| Pattern / area | Files affected | Evidence | Migration approach |
|---|---:|---|---|
| `six`, `__future__`, `distutils`, `imp`, `asyncore`, `collections.Mapping` | 0 | `git grep` over tracked Python files | None needed |
| Comment-style type hints and `class LineItem(object)` | 1 | `pricing.py:5,8,12,25-26,33-34,40-41,48-49` | Modernize conservatively; avoid altering runtime API or exception text |
| Integer-percent half-up rounding and receipt formatting | 1 | `pricing.py:40-58` | Preserve exact cents and output strings; test before edits |
| Executable `python` absent | README affected | `README.md:8-11`; `command -v python` failed | Use explicit Python 3.12/managed environment and document setup |

## Test services

`test_pricing.py:1-34` constructs local values only; no database, broker, cache, network service, Docker fixture or service connection is needed. `docker version --format '{{.Client.Version}} {{.Server.Version}}'` reported client/server 29.8.2. `DOCKER_HOST` and `TESTCONTAINERS_HOST_OVERRIDE` are present by **name**; no values were read or printed and the remote engine was not used. If future tests need containers, reach published ports through the remote engine host specified by `TESTCONTAINERS_HOST_OVERRIDE`, not localhost. `DATABASE_URL` and `REDIS_URL` were absent.

## Coverage and characterization candidates

Existing tests cover subtotal multiplication (`test_pricing.py:6-7`), code normalization and unknown codes (`:10-18`), one half-up rounding example (`:21-22`), one receipt (`:25-27`), and invalid quantity/price rejection (`:30-34`). No coverage measurement is configured. Before changing `pricing.py`, add **one** regression/characterization test on unchanged code: for example the exact `receipt([], "UNKNOWN")` output, which pins zero subtotal, zero count, unknown discount and string formatting (`pricing.py:33-58`). Other gaps to consider but not add in this phase: exact `LineItem.__repr__` (`:21-22`) and error messages (`:13-16`). These are candidates, not claims of missing behavior.

## Risks

| Risk | Severity | Likelihood | Evidence and mitigation |
|---|---|---|---|
| Dependency resolution drifts without lock/upper bound | Medium | Medium | `requirements.txt:1`; record resolved versions, consider reproducible dev dependency, rerun audit |
| Modernizing arithmetic or formatting changes observable cents/text | Medium | Low | `pricing.py:40-58`; characterize first, compare full suite and newly added test |
| README setup fails with missing `python` or system pytest | Low | High in this environment | `README.md:8-11` versus tool availability; document explicit 3.12 environment and repeat gates |
| No automated CI or lint | Low | Medium | `git ls-files` shows no tracked workflows/lint config; retain actual build/test commands and recorded evidence |

No unsupported third-party production framework, native extension, Windows-only API, or external test-service blocker was observed.

## Recommended approach / Candidate tasks

These are **recommendations, not an approved plan**; Phase 1 makes no source edits.

1. Characterize unchanged behavior with one regression test (empty/unknown-code receipt); run full build and test gates. Skill: `test-baseline-and-characterization`.
2. In-place change the Python minimum and deliberately modernize only the small scoped sample while preserving behavior and test count; align README Python 3.12 setup. Dependencies: task 1. Skills: `build-fix-loop`, `test-baseline-and-characterization`.
3. Run authoritative whole-repository Python 3.12 compilation, full pytest/JUnit and dependency audit; record exact outcomes and marker `acceptance-20261025051138-4c2f94-linux-validated` only if validated; produce final report. Dependencies: tasks 1–2. Skill: `modernization-report`.

## Open questions

None requiring a human decision for Phase 1: Python 3.12, whole small sample scope, and preservation of behavior are specified by the goal. An exact dependency pin/lock policy and whether to add CI can be considered later, not assumed approved here.
