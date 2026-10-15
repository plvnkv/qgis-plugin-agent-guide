# QGIS Plugin Agent Guide

A reusable [AGENTS.md](AGENTS.md) for AI-assisted QGIS plugin development and testing. It helps coding agents inspect an existing project, implement focused changes, test against real PyQGIS, and report what was actually verified.

## Quick start

### 1. Generate your plugin with QGIS Plugin Builder

Start by generating the plugin files in QGIS. This repository provides agent instructions; Plugin Builder provides the initial plugin code and structure.

1. Open QGIS and go to **Plugins → Manage and Install Plugins**.
2. Install the Plugin Builder version compatible with your QGIS installation:
   - [Plugin Builder 3](https://plugins.qgis.org/plugins/pluginbuilder3/) for QGIS 3.
   - [Plugin Builder 4](https://plugins.qgis.org/plugins/pluginbuilder4/) for QGIS 4 / Qt6.
   Check the selected release's compatibility before installation.
3. Open Plugin Builder, enter your plugin's name, Python package/class details, description, and other required metadata.
4. Select a template appropriate to your workflow. For parameter-driven geoprocessing, consider a Processing provider template.
5. Generate the plugin into your development directory.
6. Follow the generated setup instructions, including any resource compilation or dependency setup required by that template.

See the [Plugin Builder documentation](https://jonah-sullivan.github.io/Qgis-Plugin-Builder/) for the generator workflow. Generated files are a starting point; they still need environment verification and testing.

If your plugin already exists, use its current files and skip generation. Do not regenerate over existing work.

### 2. Copy the agent instructions into your plugin project

Download [AGENTS.md](AGENTS.md) from this repository and copy it to the root of your plugin project, then commit it with your project files.

For a simple generated project, the result may look like this:

```text
my_plugin/
├── AGENTS.md
├── metadata.txt
├── __init__.py
├── ... generated plugin modules and resources
└── ... tests and project documentation
```

If your repository contains the plugin package in a subdirectory, place `AGENTS.md` at the repository root so it covers both the package and its tests. Preserve the layout generated for your template.

If an `AGENTS.md` already exists, merge the relevant guidance instead of overwriting project-specific instructions. This repository currently provides one general-purpose version for QGIS Python plugins.

### 3. Open the project in your AI coding tool

Use an agent with access to your project files and terminal. Ensure it reads `AGENTS.md`; if your tool does not discover that filename automatically, explicitly ask it to read and follow the file.

Give the agent a concrete plugin goal, the main user workflow, and any known compatibility requirements. Start with this prompt:

```text
Read AGENTS.md and inspect the existing generated plugin files before editing.
Attempt to fill every TBD in the project profile from metadata, source code,
documentation, tests, CI, and build scripts, citing the relevant files.
Distinguish declared compatibility from actually tested support.
Ask only about missing or conflicting information that affects the work.

Verify the local QGIS/PyQGIS environment and the available test commands.
Report baseline results, including anything that could not be run.
Then help me define acceptance criteria for the first feature and implement
it as a small, testable change. Follow the development and testing workflow
in AGENTS.md and report executed evidence rather than assumed success.
```

The agent should use existing plugin files to resolve `TBD` values wherever possible. It must not infer unverified platform support from metadata, or silently turn generated defaults into owner-approved requirements.

### 4. Develop and test incrementally

For each feature, define the expected inputs, outputs, and error behavior. The agent should implement a focused change and run the relevant checks:

| Check | What it establishes |
| --- | --- |
| Pure-Python tests | Correctness of logic independent of QGIS |
| Real PyQGIS integration tests | GIS operations, providers, CRS handling, attributes, and persisted outputs |
| Qt/UI tests | Relevant widget behavior, validation, signals, and errors |
| Desktop smoke tests | Installation and the main workflow in an actual QGIS application |
| Package checks | The exact release ZIP contains and installs the required files |

Use small datasets with independently known results. Check output contents, not just whether an operation finishes without an exception. For file outputs, reopen the written data and verify it.

There is no universal test command in this guide: the agent must discover and verify commands for your project and target environment. A normal Python environment alone does not provide the native QGIS runtime. See the [PyQGIS setup documentation](https://docs.qgis.org/3.44/en/docs/pyqgis_developer_cookbook/intro.html), selecting the documentation version that matches your target.

### 5. Validate the packaged plugin before release

Build the installable plugin ZIP and install that exact artifact into a dedicated, clean QGIS test profile. Check loading, the primary workflow, error messages, and repeated enable/disable behavior. Check long-running operations and cancellation if applicable.

Headless tests do not establish desktop compatibility, and Linux results do not establish Windows or macOS support. If the agent cannot operate the QGIS desktop, execute the smoke checklist yourself and record the results.

## What AGENTS.md covers

- Project discovery and evidence-based completion of `TBD` fields.
- Version-aware environment setup and official API verification.
- Separation of pure logic, GIS services, Processing, UI, and lifecycle code.
- Focused implementation, regression testing, and review.
- CRS, units, geometry, attributes, persistence, and failure cases.
- QGIS background-task constraints and lifecycle cleanup.
- CI, packaging, desktop smoke testing, and release evidence.
- Honest **PASS**, **FAIL**, and **NOT RUN** reporting.

Adapt the profile and workflow to your project. Keep documentation lightweight, preserve explicit requirements, and do not label a plugin release validated while required checks remain unexecuted.

## Repository contents

- [AGENTS.md](AGENTS.md) — copy this file into your plugin project.
- [README.md](README.md) — setup and usage instructions for this guide.

## References

- [QGIS documentation](https://docs.qgis.org/)
- [PyQGIS API](https://qgis.org/pyqgis/)
- [Official QGIS plugin repository](https://plugins.qgis.org/)
- [Plugin Builder source](https://github.com/jonah-sullivan/Qgis-Plugin-Builder)

This is an independent guide, not an official QGIS project. The guide itself does not certify a plugin or a QGIS version as tested.
