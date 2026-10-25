# Modernization report: acceptance-20261025051138-4c2f94 linux modernization

Outcome: SUCCEEDED - 2 of 3 tasks checkpoint-completed; T003 validation passed and this report awaits its checkpoint; build GREEN; tests 7/7 passing (baseline 6/6).

The scoped goal is met: Python 3.12 is declared, existing pricing behavior is unchanged, one regression test was added, and the final full gates passed after the last source/configuration edit. Validation marker: `acceptance-20261025051138-4c2f94-linux-validated`. The marker reflects the authoritative final build and test results in [T003's progress note](progress/T003.md), not a claim that T003 has already been checkpointed.

## Summary

- Goal: modernize the small synthetic pricing sample to Python 3.12 while preserving behavior, adding one regression test and documenting exact validation.
- Scenario: none specified; path scope: the small sample and its modernization artifacts (run state).
- Strategy: in-place metadata/documentation update rather than rewriting already-passing pricing code ([assessment](assessment.md), [T002](progress/T002.md)).
- Branch: `acceptance-20261025051138-4c2f94-linux-modernization` based on `main`; two checkpoint commits at report-writing time (`git log --oneline main..HEAD`). The approved three-task plan is recorded in run state; there is no `plan.md` on disk.

## Versions

| Component | Before | After | Support/LTS status (source; checked on) |
|---|---|---|---|
| Declared Python minimum | `>=3.10` | `>=3.12` | Python 3.10 end of life 2026-10-01; Python 3.12 in security maintenance through October 2028, per [Python developer guide](https://devguide.python.org/versions/), checked 2026-10-25 in [assessment](assessment.md). This is a *minimum-version declaration*, not evidence of a former deployed interpreter. |
| Interpreter used for baseline and final gates | Python 3.12.3 | Python 3.12.3 | Installed version and target verified in [assessment](assessment.md) and [T003](progress/T003.md); no interpreter replacement was performed. |
| pytest (test library) | `pytest>=7`; resolved 9.1.1 | Unchanged specification; final resolved 9.1.1 | pytest 9.0+ supports Python 3.10+, per [pytest compatibility documentation](https://docs.pytest.org/en/stable/backwards-compatibility.html#python-version-support), checked 2026-10-25 in [assessment](assessment.md); exact support end date unknown. |
| pluggy (transitive test library) | Resolved 1.7.0 | Final test run reported 1.7.0 | Independent support window unverified ([assessment](assessment.md), [T003](progress/T003.md)). |
| Framework / production third-party packages | None declared | None declared | No framework support claim; `pricing.py` uses standard-library `typing` ([assessment](assessment.md)). |

## Tasks

| Key | Title | Status at report-writing time | Checkpoint commit | Result |
|---|---|---|---|---|
| T001 | Adding one pricing characterization test | Completed | `63c165fd46e984fab5b89bca0ed1896dc01261fe` | Added one empty-receipt/unknown-code regression test ([note](progress/T001.md)). |
| T002 | Declaring and documenting Python 3.12 | Completed | `3099098faf94d3727b754dab849f9445959ba0a7` | Raised Python minimum and documented runnable uv instructions ([note](progress/T002.md)). |
| T003 | Finalizing validation and report | In progress; final validation complete, report written, checkpoint pending | None yet | Final whole-sample gates GREEN; reporting handoff in [note](progress/T003.md). |

No failed or skipped tasks appear in the approved run-state ledger. At report-writing time `git diff --stat main...HEAD` shows changes to `README.md`, `pyproject.toml`, `test_pricing.py`, [assessment](assessment.md), and [T001](progress/T001.md)/[T002](progress/T002.md) notes; the T003 note and this report are not yet checkpointed. `pricing.py` was not edited.

## Validation

| Gate | Exact final command | Baseline | Final after T002 |
|---|---|---|---|
| Build (Python syntax compilation; no package build configured) | `TZ=UTC LANG=C.UTF-8 PYTHONPYCACHEPREFIX=.modernization-scratch/pycache python3.12 -m compileall -q pricing.py test_pricing.py` | Exit 0, no warnings ([assessment](assessment.md)) | Exit 0, 0 warnings; GREEN ([T003](progress/T003.md), authoritative final BuildValidator verdict) |
| Full tests | `mkdir -p .modernization-scratch/T003 && TZ=UTC LANG=C.UTF-8 PYTHONPYCACHEPREFIX=.modernization-scratch/pycache UV_CACHE_DIR=.modernization-scratch/uv-cache uv run --no-project --python /usr/bin/python3.12 --with-requirements requirements.txt python -m pytest -rA --junitxml=.modernization-scratch/T003/pytest.xml` | 6 passed, 0 failed, 0 skipped; 0 warnings ([assessment](assessment.md)) | Exit 0; XML: 7 passed, 0 failed, 0 errors, 0 skipped; 0 warnings; GREEN ([T003](progress/T003.md), authoritative final BuildValidator verdict) |
| Lint | Not configured | Not run | Not run ([assessment](assessment.md)) |

- Added characterization/regression test: `test_pricing.py::test_receipt_with_no_items_and_unknown_code` checks the exact output `0 item(s), discount 0.00, total 0.00` for `receipt([], "UNKNOWN")` ([T001](progress/T001.md)). Existing six tests still pass; no other tests were added.
- Approved test exclusions: none (run state); no tests skipped in the final XML. The full-branch CodeReviewer verdict was **approve** (authoritative review result provided for this report).
- Generated XML/caches are local, excluded scratch outputs, not tracked artifacts ([T003](progress/T003.md)).

## Dependencies and security

| Package | Before | After | Change | Reason |
|---|---|---|---|---|
| pytest | `>=7` in `requirements.txt` | `>=7` | None | Keep existing working test dependency; no lock or bound added ([T002](progress/T002.md), `git diff main...HEAD -- '*requirements*.txt'`). |

No packages were added, removed, upgraded or replaced by the branch. The [assessment](assessment.md) records a **baseline-only** `pip-audit` of resolved dependencies: 0 known critical, 0 high, 0 medium and 0 low findings. **After-change vulnerability counts: unknown; no post-change scan was run.** No finding was recorded as open at baseline; the unchanged, unbounded dependency specification is not evidence that all future resolutions are safe. Re-audit after dependency resolution changes.

## Behavior and compatibility changes

| Change | Consumer/operator impact | Decision and rationale | Test coverage |
|---|---|---|---|
| Raise `requires-python` from `>=3.10` to `>=3.12` in `pyproject.toml` | Package metadata excludes Python below 3.12; deployment interpreter must satisfy the new minimum. | Target requested in run goal; Python 3.12 baseline already passed ([assessment](assessment.md), [T002](progress/T002.md)). | Full seven-test suite on Python 3.12.3 ([T003](progress/T003.md)); behavior on older interpreters is not claimed. |
| Replace README's bare-`python`/pip setup with explicit Python 3.12 and uv commands | Operators need Python 3.12 and uv to follow the documented isolated test invocation. | Bare `python` and system pytest were absent locally; new commands were exercised ([assessment](assessment.md), [T002](progress/T002.md)). | README commands and exact gates passed ([T002](progress/T002.md), [T003](progress/T003.md)). |
| Leave pricing implementation and `pytest>=7` unchanged | No intentional change to pricing outputs, arithmetic, formatting or dependency specification. | Already passes Python 3.12; preserve behavior and scope (approved plan summary, [T001](progress/T001.md), [T002](progress/T002.md)). | Six existing tests plus the new exact-receipt regression test; no claim of exhaustive behavior coverage. |

The base-branch diff shows no edits to `pricing.py`, routes, serialization, ports, configuration keys, encodings or logging; no such runtime behavior change is claimed.

## CI, deployment and configuration

- Workflow files changed: none; no tracked CI workflow was present ([assessment](assessment.md), `git diff main...HEAD -- .github/workflows Dockerfile* '*.yml'`). No workflow-change confirmation is needed for this branch.
- Runtime prerequisite for any future deployment: Python >=3.12; installed runtime and deployment compatibility outside the local validation are **not verified**. uv is used for local test setup, not established as a production prerequisite.
- Configuration/environment variables added or renamed: none. Validation sets `TZ`, `LANG`, `PYTHONPYCACHEPREFIX` and `UV_CACHE_DIR` for its local commands; these are not new application settings.
- Container files, images, ports and health endpoints changed: none; no deployment performed ([assessment](assessment.md), branch diff).

## Risks and follow-ups

| Item | Severity | Recommendation |
|---|---|---|
| Unbounded `pytest>=7`, no lockfile; future resolution may differ | Medium ([assessment](assessment.md)) | Decide whether to pin/lock test dependencies in a separately approved change; re-run tests and dependency audit. |
| No configured lint or automated CI | Low ([assessment](assessment.md)) | Review whether to add gates in a future scoped change; do not edit workflows under this run. |
| Future deployment Python version and compatibility | Unknown | Operators should confirm Python >=3.12 in their own environments before deployment; no deployment was validated here. |
| T003 checkpoint not yet present at report writing | Process follow-up | Complete the task checkpoint through the run workflow after reviewing this report; do not attribute a commit to T003 prematurely. |

## Open questions

- Should test dependencies be locked and CI/lint added later? Recommendation: decide in a separately scoped follow-up; none is authorized in this run.

## How to verify locally

From the repository root with Python 3.12 at `/usr/bin/python3.12` and uv available, run the exact final gates (commands and outcomes sourced from [T003](progress/T003.md)):

```sh
TZ=UTC LANG=C.UTF-8 PYTHONPYCACHEPREFIX=.modernization-scratch/pycache python3.12 -m compileall -q pricing.py test_pricing.py
mkdir -p .modernization-scratch/T003 && TZ=UTC LANG=C.UTF-8 PYTHONPYCACHEPREFIX=.modernization-scratch/pycache UV_CACHE_DIR=.modernization-scratch/uv-cache uv run --no-project --python /usr/bin/python3.12 --with-requirements requirements.txt python -m pytest -rA --junitxml=.modernization-scratch/T003/pytest.xml
```

There is no configured application run command or health URL; `pricing.py` is a single-file library ([assessment](assessment.md)). Next, review the branch and push it from Futuristic; this workspace has not pushed or opened a pull request.

## Artifacts

- [Assessment and baseline](assessment.md)
- Approved plan: run state (no `plan.md` exists on disk)
- Progress: [T001](progress/T001.md), [T002](progress/T002.md), [T003](progress/T003.md)
