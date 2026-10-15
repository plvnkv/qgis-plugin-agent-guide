# QGIS Plugin Development and Testing

This file guides AI coding agents working in this repository. Keep the process proportional to the project. Prefer small, reviewable changes and executed evidence over extensive paperwork.

## 1. Establish the project context

Before implementing the first feature, inspect the repository and complete the following profile using verified facts and the owner's requirements. `TBD` means unknown, not permission to choose a product requirement silently. An existing repository may document these values elsewhere; link to that source instead of duplicating it.

If plugin files already exist, first attempt to resolve every `TBD` from those files before asking the owner. Inspect `metadata.txt`, Python entry points and modules, Processing providers and algorithms, UI files, README/documentation, dependency files, tests, CI configuration, and build/packaging scripts as applicable. Fill supported values into the profile or link to their authoritative repository source, citing the relevant file. Distinguish declared compatibility, observed implementation behavior, and actually tested support: metadata or CI configuration alone does not prove that a version or platform passes. Verify discovered commands before describing them as working. Leave unresolved values as `TBD`, identify conflicting or incomplete evidence, and ask only for information that cannot be reliably established from the project. Do not overwrite explicit owner requirements with assumptions inferred from existing code.

| Setting | Value |
| --- | --- |
| Plugin name and Python package | TBD |
| Purpose and primary user workflow | TBD |
| Supported QGIS versions | TBD — exact versions or explicit range |
| Supported operating systems | TBD |
| Python and Qt versions provided by each target environment | TBD — inspect the environment |
| Input/output formats, providers, geometry types | TBD |
| CRS and measurement policy | TBD |
| Expected data size and performance requirements | TBD if relevant |
| External services and dependencies | TBD, or explicitly none |
| Requirements and acceptance criteria location | TBD |
| Test commands and environment setup location | TBD |
| Packaging command and artifact location | TBD |

Ask focused questions when missing information changes observable behavior, compatibility, data handling, or scope. Continue independent inspection and setup while awaiting answers. Make routine, reversible implementation decisions autonomously and document material assumptions.

Do not assume a QGIS release is current, supported, or compatible based on memory. Check the official documentation and API for the selected version. Do not assume compatibility across QGIS major versions, Qt versions, or operating systems.

## 2. Working agreements

- Read existing project instructions, requirements, code, tests, and dependency configuration before editing.
- Preserve unrelated user changes. Do not overwrite or reset work to simplify a task.
- Do not invent features, silently broaden scope, or change acceptance criteria to make tests pass.
- Keep requirements lightweight: a README section or an existing issue is sufficient for a small plugin.
- Give requirements and acceptance criteria stable identifiers when tracking multiple features. Connect each meaningful change to its requirement, test, and evidence; do not create redundant tracking documents.
- Use official QGIS/Qt documentation and upstream source to verify unfamiliar APIs. Check that the API exists in the minimum supported version.
- Never treat a proposed test, generated code, a screenshot, or self-review as proof of a passing test.
- Do not install dependencies, alter the user's normal QGIS profile, or modify real GIS datasets merely to make testing convenient. Use an isolated development environment, a dedicated test profile, and disposable data copies. Follow the user's authorization for environment changes.
- Never include credentials, private connection strings, or confidential geodata in fixtures, logs, or commits.

## 3. Environment before implementation

Establish a reproducible development/test environment with real QGIS libraries before relying on integration results.

1. Inspect available QGIS installations and their Python, Qt, GDAL, GEOS, and PROJ versions where relevant.
2. Verify that the selected interpreter can import `qgis.core` and initialize a `QgsApplication`.
3. Verify required providers and Processing algorithms are available. An import alone is not enough.
4. Record the interpreter or launcher, environment variables, dependency versions, and exact working commands in the repository's setup documentation.
5. Run the existing relevant checks and distinguish baseline failures from regressions.

Do not assume a normal virtual environment or installing pytest provides PyQGIS. Resolve the QGIS library paths and native dependencies for the actual operating system. Do not copy platform-specific paths without checking them.

For automated Linux testing, use a reproducible environment with a pinned QGIS version, such as a verified container image. Pin the image tag or digest; do not rely on `latest`. Configure a virtual display or Qt offscreen mode only as supported by that environment. A headless pass does not prove desktop integration.

If QGIS is unavailable, continue pure-Python work and static checks where useful. Mark GIS and desktop validation as **NOT RUN** with the reason. Do not replace QGIS with mocks and report an integration pass.

## 4. Architecture

Separate responsibilities where useful; do not impose unnecessary layers on a small plugin:

- **Pure logic:** validation and calculations that do not require QGIS or Qt.
- **GIS services:** geometry, CRS transformations, providers, and file operations using PyQGIS.
- **Processing algorithms:** explicit inputs, parameters, outputs, context, and feedback.
- **UI:** dialogs, actions, messages, and selection of user inputs.
- **Plugin lifecycle:** registration and cleanup of actions, signals, providers, and resources.

Prefer a Processing algorithm for parameter-driven geoprocessing when it fits the user's workflow. Keep business rules out of button handlers. Use the `qgis.PyQt` compatibility namespace for Qt imports and verify APIs against the supported environment.

Keep `initGui()` and `unload()` consistent: remove registered UI elements, disconnect owned signal connections, and unregister owned providers/resources. Check repeated enable/disable cycles for duplicated actions or callbacks.

For background work, follow QGIS task threading rules. Never access GUI objects or main-thread-owned QObjects such as project or layer objects from a worker. Prepare independent data on the owning thread, respect object lifetimes, and deliver results safely. Handle errors and cancellation without presenting partial outputs as successful results.

## 5. Development loop

For each feature or bug fix:

1. Identify the requirement and observable acceptance criteria. State inputs, outputs, error behavior, and relevant GIS assumptions.
2. Inspect nearby code and tests. For a bug, reproduce it and identify the cause before changing behavior.
3. Define an independent expected result. Add a focused regression or behavior test for meaningful logic changes; avoid tests that merely mirror the implementation.
4. Implement the smallest coherent change using existing project conventions.
5. Run relevant checks in the correct environment, inspect failures, and fix the cause. Do not weaken assertions or skip failing checks to manufacture success.
6. Review the diff for accidental scope changes, lifecycle problems, data loss, threading errors, and unsupported APIs.
7. Record executed checks, results, and limitations. Update user documentation when behavior or setup changes.

For documentation-only or similarly low-impact changes, use proportionate content checks rather than inventing runtime tests. Run broader tests when the change affects shared code or reveals unresolved risks.

## 6. Test layers

### Pure-Python tests

Test independent validation, parsing, rules, and calculations without launching QGIS where possible. Mock external services at their boundaries, with explicit success and failure cases. Do not use mocked geometry or CRS operations as evidence of GIS correctness.

### Real PyQGIS integration tests

Use a compatible test harness, such as QGIS's `qgis.testing` utilities or a verified compatible pytest integration. Initialize the application once per appropriate test scope, register required providers, and clean up projects, layers, settings, and temporary files between tests.

For Processing tests, initialize Processing and register the plugin's provider explicitly where required. Verify algorithm discovery, parameter validation, execution, and output contents.

Do not assume a stubbed `iface` is the real QGIS desktop interface. It can support focused tests but cannot establish full application integration.

### Qt/UI tests

Where useful, test dialog validation, enabled states, signals, error messages, and cancellation. Use event-based waits with bounded timeouts rather than arbitrary sleeps. Keep UI tests separate from GIS result assertions so failures are diagnosable.

### Desktop smoke tests

Run the packaged plugin in a clean, dedicated QGIS profile on the target platform. Verify installation, loading, the primary workflow, user-visible errors, output loading, and enable/disable cleanup. Test cancellation and responsiveness when applicable.

Desktop automation may perform these steps if available. Otherwise provide a concise manual checklist and report it as **NOT RUN** until someone actually executes it. Linux headless testing does not establish Windows or macOS compatibility.

## 7. GIS correctness checklist

Select the cases relevant to the feature; do not add unrelated requirements.

- **CRS and units:** source/output CRS, geographic versus projected coordinates, transform context, and planar versus ellipsoidal measurements. Never silently interpret degrees as meters or choose a projection without a defined policy.
- **Geometry:** expected type, validity, empty/null geometries, single/multipart inputs, and Z/M dimensions where supported. Make invalid-geometry handling explicit.
- **Attributes:** schema, field types, null values, encoding, identifiers, and preservation rules.
- **Results:** feature count, meaningful spatial properties, expected values, and deterministic ordering only when ordering is part of the contract.
- **Persistence:** write to a temporary output, reopen it through the intended provider, and validate contents, CRS, schema, and geometry.
- **Failures:** unsupported inputs, missing fields, unavailable providers, unwritable paths, existing-output policy, and failed external requests where applicable.
- **Scale:** representative data sizes and a defined performance target if required. Record environment and dataset size for measurements.
- **Data protection:** confirm input data remains unchanged unless editing it is explicitly part of the feature. Test rollback or partial-output cleanup when relevant.

Use tiny synthetic datasets with known results for most correctness tests. Add representative, shareable fixtures only where they cover behavior synthetic data misses. Document fixture provenance and expected results.

Choose numerical tolerances based on units, precision, and the algorithm. Use appropriate topological or coordinate comparisons instead of raw file bytes or WKT equality when serialization differences are irrelevant. Do not generate the expected answer using the same implementation under test.

## 8. CI and packaging

Automate the verified local checks in CI when the repository uses CI. Keep pure-Python checks and QGIS-dependent checks distinguishable. Test the minimum supported QGIS version and another supported target when applicable; expand the platform matrix according to the compatibility promise.

Record exact commands only after verifying them. Do not leave invented commands presented as working setup instructions. Preserve reports and useful logs as CI artifacts; capture diagnostics for native crashes and timeouts.

Before release:

- Check plugin metadata, entry points, resource availability, and declared QGIS compatibility against the target versions.
- Build the installable ZIP with the expected top-level plugin package directory.
- Inspect its contents: include required modules/resources and exclude credentials, caches, local environments, and unnecessary test data.
- Install that exact ZIP in a clean test profile and run the desktop smoke checklist.
- Ensure release notes and installation instructions match actual behavior.

Do not publish a release or upload to the plugin repository without user authorization. A successful source-tree test is not proof that the packaged artifact installs correctly.

## 9. Evidence and completion

Use these statuses accurately:

- **PASS:** the specified check ran and its assertions succeeded.
- **FAIL:** the check ran and failed; include the relevant failure and impact.
- **NOT RUN:** the check was not executed; include the reason and what is needed.

A skipped check is not a pass. A zero exit code with no intended tests collected is not a pass. A passing subset must not be described as a fully passing suite.

For each meaningful validation run, record:

| Field | Required information |
| --- | --- |
| Scope | Requirement/acceptance criterion and test or scenario |
| Revision | Commit or a clear description of the tested working tree |
| Environment | OS, QGIS, Python, Qt, relevant dependencies |
| Execution | Exact command or desktop steps, working directory, date |
| Result | PASS / FAIL / NOT RUN, test counts and skips where available |
| Evidence | Report/log path or concise captured result |
| Limitations | Untested versions, platforms, GUI flows, or datasets |

Keep this in the existing task/PR report or a lightweight repository report. Do not create a large documentation framework solely to satisfy this table.

The final handoff must state what changed, what actually passed, and what remains unverified. Distinguish **implementation complete** from **release validated**. Do not claim release readiness while required acceptance or packaging checks remain failed or unexecuted.

## 10. Official references

Select documentation matching the project's target version; these links are entry points, not a mandate to use a particular release.

- QGIS documentation: https://docs.qgis.org/
- Python API: https://qgis.org/pyqgis/
- Plugin structure (3.44 example): https://docs.qgis.org/3.44/en/docs/pyqgis_developer_cookbook/plugins/plugins.html
- Standalone PyQGIS setup (3.44 example): https://docs.qgis.org/3.44/en/docs/pyqgis_developer_cookbook/intro.html
- Processing plugins (3.44 example): https://docs.qgis.org/3.44/en/docs/pyqgis_developer_cookbook/processing.html
- Background tasks (3.44 example): https://docs.qgis.org/3.44/en/docs/pyqgis_developer_cookbook/tasks.html
- Upstream test utilities: https://github.com/qgis/QGIS/tree/master/python/testing — inspect the matching release branch when using these as a reference.
