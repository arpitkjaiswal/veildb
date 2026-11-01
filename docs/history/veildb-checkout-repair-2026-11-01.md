# veildb checkout repair — 2026-11-01

## What was repaired

The initial upload omitted 15 runtime modules, the test suite, documentation, and
other supporting files. Importing the public `Project` API failed because its
module was absent. Internal imports also referenced missing numeric, JSONB,
query, migration, and rotation implementations.

The existing runtime, harnesses, dependency declarations, and lockfile exactly
matched the documented upstream at
[`8a5d5060b7aacf5cb3bae6937946fb752cf9ca29`](https://github.com/Debrato2005/cryptalis/tree/8a5d5060b7aacf5cb3bae6937946fb752cf9ca29).
Only missing files were restored: 1,849 files, including all 15 omitted runtime
modules and 273 test/fixture files. Existing implementation files were preserved.
This repairs an incomplete checkout; it introduces no new encryption design or
capability admission.

Current maintainer metadata uses `arpitkjaiswal`, and package project links point
to this repository. The original MIT copyright notice and historical evidence
remain intact. The Python distribution, imports, CLI, and serialized format
identifiers retain the `cryptalis` namespace.

## Completed local checks

| Check | Observed outcome | Receipt |
| --- | --- | --- |
| Complete default source suite | 3,953 test calls and 18 subtests pass; 25 call, 41 setup, and nine collection skips; no failures | [Source receipt](../_reset/veildb-source-20261101.json) |
| Public API and module imports | `Project` and all 43 non-entry-point submodules import; CLI help starts | Current-instance diagnostic |
| Repeated wheel and source-archive builds | Matching artifact hashes across two isolated builds; wheel RECORD, metadata, source imports, and SBOM inspection pass | [Package receipt](../_reset/veildb-package-20261101.json) |
| Clean wheel installation | Hashed dependencies install consistently; 50 installed files match the inspected wheel and its CLI starts | [Installed-file inspection](../_reset/veildb-wheel-install-20261101.json) |
| Installed public API and CF1 tests | 111 passed, no skips | [Installed test receipt](../_reset/veildb-installed-20261101.json) |
| Installed DATE backup recovery with original public slots | One export and one restore test pass on separately initialized clusters, including rotations and package-free exit | [Export](../_reset/veildb-date-export-20261101.json), [restore](../_reset/veildb-date-restore-20261101.json) |
| Reusable startup helper | Six online-protection integration tests pass after fresh role probes | [Startup receipt](../_reset/veildb-startup-20261101.json) |
| Optional research helper | Three existing unittest cases pass in the separate Crawl4AI 0.9.5 environment | Current-instance test output |

Application dependencies were installed from the unchanged lockfile with all
extras. The database checks use PostgreSQL 16.15 and Python 3.12.3 in Ubuntu
24.04, four workers for the full source run, and the mandatory credential-probing
wrapper. Host import, package-build, and research checks use Python 3.12.14 and
uv 0.12.23. Each database run creates a disposable local cluster; no existing
database or user-provided database credentials are used.

The default source suite completed in 668.56 seconds and reported 67
deprecation warnings. Its terminal reports 75 skips across all phases. The raw
`counts.passed` value of 3,971 includes the 18 subtests; it is not 3,971 distinct
test items. The separate `pytest_report_counts` fields retain those boundaries.
Optional external application snapshots, prior-reader interpreters, scanner
tooling, the SQLModel-absent control in an all-extras environment, and opt-in
qualification profiles remain outside that passing default selection. No
assertion or skip condition was changed to obtain these results.

The first research-helper run in the application interpreter failed because its
optional Crawl4AI dependency was absent. Installing its pinned version in the
documented separate environment resolved that setup failure; application runtime
dependencies were not expanded.

## Reproduce the source check

On the documented native PostgreSQL 16 development platform, from the repository
root:

```sh
uv sync --frozen --all-extras
run_root=$(mktemp -d /tmp/veildb-check.XXXXXX)
uv run --frozen --all-extras python scripts/online_qualification.py \
  --workdir "$run_root/source" -- tests -q -n 4 --dist loadfile
```

In this managed cloud environment, the saved installation recipe creates
`/workspace/.setup/veildb/run-source-check.sh` for the same selection inside the
tested Ubuntu/PostgreSQL image. See [local development](../local-development.md)
for prerequisites and the separate qualification profiles.

## Remaining verification limits

The frozen dependency audit could not reach `api.osv.dev/v1/querybatch`. A
separate connectivity check confirmed a proxy CONNECT 403. The running
environment's allowlist omits this host. A targeted domain addition was saved in
the environment draft; saving the draft does not apply the network change or
complete the audit. Review/save the environment settings and publish the prepared
environment, then rerun the audit once the policy is applied.

The tested installation recipe and startup instructions were saved in the
environment's `install_script` and `start_skill` fields. Installation was checked
for repeatability, and the startup command was exercised with the six-test
integration selection. Publication and restoration in a new task have not been
verified. Docker images must be rebuilt if unavailable after restoration.

These local checks do not qualify live cloud-provider custody, external
application fixtures, production operation, or independent security review.
Historical upstream receipts retain their original revision and environment
scope. The seven production gates remain UNKNOWN.
