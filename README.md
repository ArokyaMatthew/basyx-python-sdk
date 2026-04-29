# Eclipse BaSyx Python SDK

[![CI](https://github.com/eclipse-basyx/basyx-python-sdk/actions/workflows/ci.yml/badge.svg)](https://github.com/eclipse-basyx/basyx-python-sdk/actions/workflows/ci.yml)
[![PyPI Version](https://img.shields.io/pypi/v/basyx-python-sdk)](https://pypi.org/project/basyx-python-sdk/)
[![Conda Version](https://img.shields.io/conda/vn/conda-forge/basyx-python-sdk)](https://anaconda.org/conda-forge/basyx-python-sdk)
[![PyPI Downloads](https://img.shields.io/pypi/dm/basyx-python-sdk)](https://pypi.org/project/basyx-python-sdk/)
[![Python Version](https://img.shields.io/pypi/pyversions/basyx-python-sdk)](https://pypi.org/project/basyx-python-sdk/)
[![License](https://img.shields.io/github/license/eclipse-basyx/basyx-python-sdk)](LICENSE)

The Eclipse BaSyx Python SDK is a production-grade Python implementation of the
[Asset Administration Shell (AAS)](https://industrialdigitaltwin.org/en/content-hub/aasspecifications)
for Industry 4.0 systems. It lets you model, serialize, validate, store, and serve AAS data
entirely in Python — from rapid prototyping on a laptop to running a specification-compliant
HTTP server in Docker.

The project is part of the [Eclipse BaSyx](https://www.eclipse.org/basyx/) middleware
framework, developed under the umbrella of the Eclipse Foundation.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Specification Compliance](#specification-compliance)
- [Getting Started](#getting-started)
- [Repository Structure](#repository-structure)
- [Tutorials](#tutorials)
- [Documentation](#documentation)
- [FAQ](#faq)
- [Release Schedule](#release-schedule)
- [Contributing](#contributing)
- [License](#license)

---

## Project Overview

This mono-repository contains three self-contained Python packages that cover different
aspects of working with Asset Administration Shells:

| Package | Purpose | Details |
|---------|---------|---------|
| **[SDK](./sdk)** | Core library — AAS metamodel, serialization, storage | [SDK README](./sdk/README.md) |
| **[Server](./server)** | Spec-compliant AAS HTTP/REST server (Docker) | [Server README](./server/README.md) |
| **[Compliance Tool](./compliance_tool)** | CLI checker for AAS file compliance | [Compliance Tool README](./compliance_tool/README.md) |

### SDK

The SDK (`basyx-python-sdk` on PyPI) is the core of this project. It provides:

- **AAS Metamodel** — full Python object model of the AAS metamodel (Part 1)
- **Serialization** — read and write AAS data as JSON, XML, or AASX package files
- **Backend Storage** — persist AAS objects in CouchDB or as local JSON files, with an
  extensible backend interface
- **Compliance Checking** — validate AAS JSON and XML files against the official schemas
- **Experimental RDF Support** — serialization to RDF is available on the
  [Experimental/Adapter/RDF](https://github.com/eclipse-basyx/basyx-python-sdk/tree/Experimental/Adapter/RDF/basyx/aas/adapter/rdf)
  branch (see [#308](https://github.com/eclipse-basyx/basyx-python-sdk/pull/308) for context)

### Server

A Docker image that exposes a specification-compliant HTTP/REST API (AAS Part 2), currently
implementing the **Asset Administration Shell Repository** and **Submodel Repository** service
interfaces. It can serve AAS data from AASX, JSON, or XML files, optionally with persistent
storage via the Local-File Backend.

### Compliance Tool

A command-line utility for checking whether AAS JSON, XML, or AASX files conform to the
official schema. Useful for CI pipelines, data validation, and interoperability testing.

---

## Specification Compliance

> **Note:** The SDK version number is independent of the supported AAS specification versions.

The table below lists the AAS specifications implemented by the
[current release](https://github.com/eclipse-basyx/basyx-python-sdk/releases/latest).
For older specification support, consult the [prior releases](https://github.com/eclipse-basyx/basyx-python-sdk/releases) —
each release has a similar table in its notes.

| Specification | Version |
|---|---|
| Part 1: Metamodel | [v3.0.1 (01001-3-0-1)](https://industrialdigitaltwin.org/wp-content/uploads/2024/06/IDTA-01001-3-0-1_SpecificationAssetAdministrationShell_Part1_Metamodel.pdf) |
| Schemata (JSONSchema, XSD) | [v3.0.8 (IDTA-01001-3-0-1_schemasV3.0.8)](https://github.com/admin-shell-io/aas-specs/releases/tag/IDTA-01001-3-0-1_schemasV3.0.8) |
| Part 2: API | [v3.0 (01002-3-0)](https://industrialdigitaltwin.org/en/wp-content/uploads/sites/2/2023/06/IDTA-01002-3-0_SpecificationAssetAdministrationShell_Part2_API_.pdf) |
| Part 3a: Data Specification IEC 61360 | [v3.0 (01003-a-3-0)](https://industrialdigitaltwin.org/wp-content/uploads/2023/04/IDTA-01003-a-3-0_SpecificationAssetAdministrationShell_Part3a_DataSpecification_IEC61360.pdf) |
| Part 5: Package File Format (AASX) | [v3.0 (01005-3-0)](https://industrialdigitaltwin.org/wp-content/uploads/2023/04/IDTA-01005-3-0_SpecificationAssetAdministrationShell_Part5_AASXPackageFileFormat.pdf) |

---

## Getting Started

### Requirements

- Python 3.10 or newer

### Installation

Install the SDK from **PyPI**:

```bash
pip install basyx-python-sdk
```

Or from **conda-forge**:

```bash
conda install -c conda-forge basyx-python-sdk
```

To install directly from the latest development branch:

```bash
pip install git+https://github.com/eclipse-basyx/basyx-python-sdk@main#subdirectory=sdk
```

> **Tip:** Consider using a virtual environment (`python -m venv .venv`) to isolate
> project dependencies.

### Quick Example

Create a `Submodel` with a `Property` and write it to an XML file:

```python
from basyx.aas import model
from basyx.aas.adapter.xml import write_aas_xml_file

# Create a Submodel
submodel = model.Submodel('https://example.org/submodels/ExampleSubmodel')

# Add a Property with a semantic reference
submodel.submodel_element.add(
    model.Property(
        id_short='MaxTemperature',
        value_type=model.datatypes.Double,
        value=42.5,
        semantic_id=model.ExternalReference(
            (model.Key(
                type_=model.KeyTypes.GLOBAL_REFERENCE,
                value='https://example.org/properties/MaxTemperature'
            ),)
        )
    )
)

# Serialize to XML
data: model.DictIdentifiableStore[model.Identifiable] = model.DictIdentifiableStore()
data.add(submodel)
write_aas_xml_file(file='ExampleSubmodel.xml', data=data)
```

For more involved workflows — creating full AAS shells, navigating submodels, working with
AASX packages — see the [Tutorials](#tutorials) section below.

---

## Repository Structure

```
basyx-python-sdk/
├── sdk/                    # Core Python SDK (basyx-python-sdk on PyPI)
│   ├── basyx/aas/
│   │   ├── model/          # AAS metamodel as Python classes
│   │   ├── adapter/        # JSON, XML, AASX serialization adapters
│   │   ├── backend/        # CouchDB, local-file storage backends
│   │   ├── util/           # Utility functions (traversal, identification)
│   │   └── examples/       # Tutorials and example data
│   ├── docs/               # Sphinx documentation source
│   └── pyproject.toml
├── server/                 # AAS HTTP/REST server (Docker)
│   ├── app/                # Server application code
│   ├── docker/             # Dockerfiles and entrypoint scripts
│   ├── example_configurations/
│   └── pyproject.toml
├── compliance_tool/        # CLI compliance checker
│   ├── aas_compliance_tool/
│   └── pyproject.toml
├── .github/workflows/      # CI/CD pipeline definitions
├── CONTRIBUTING.md
├── LICENSE                  # MIT License
└── NOTICE
```

---

## Tutorials

The SDK ships with step-by-step tutorials in `sdk/basyx/aas/examples/`:

| Tutorial | What You Will Learn |
|---|---|
| [Create a Simple AAS](./sdk/basyx/aas/examples/tutorial_create_simple_aas.py) | Build an Asset Administration Shell with an Asset and a Submodel from scratch |
| [Navigate Submodels](./sdk/basyx/aas/examples/tutorial_navigate_aas.py) | Traverse AAS Submodels using IdShorts and IdShortPaths |
| [Object Storage](./sdk/basyx/aas/examples/tutorial_storage.py) | Manage many AAS objects with ObjectStores and resolve references |
| [Serialization & Deserialization](./sdk/basyx/aas/examples/tutorial_serialization_deserialization.py) | Read and write AAS data as JSON and XML |
| [AASX Packages](./sdk/basyx/aas/examples/tutorial_aasx.py) | Export AAS shells with related objects and auxiliary files to AASX packages |
| [CouchDB Backend](./sdk/basyx/aas/examples/tutorial_backend_couchdb.py) | Store and retrieve AAS objects in a CouchDB document database |

---

## Documentation

Full API documentation is hosted on Read the Docs:

**[basyx-python-sdk.readthedocs.io](https://basyx-python-sdk.readthedocs.io)**

---

## FAQ

**What Python versions are supported?**

Python 3.10 through 3.12. The CI matrix tests against both ends of that range.
Supported versions are checked automatically to ensure none are past end-of-life.

**What is the relationship between the SDK version and the AAS specification version?**

They are independent. The SDK follows [semantic versioning](https://semver.org/) for its own
API surface. The specification versions it implements are listed in the
[Specification Compliance](#specification-compliance) table and in each GitHub release's notes.

**Can I run the server without Docker?**

Yes, for debugging purposes. See the
[Server README](./server/README.md#running-without-docker-debugging-only) for instructions.
This mode is not suitable for production.

**How do I validate my AAS files?**

Use the Compliance Tool:

```bash
pip install aas-compliance-tool
aas-compliance-check --help
```

See the [Compliance Tool README](./compliance_tool/README.md) for detailed usage.

**Where can I find the Conda package?**

The SDK is published on
[conda-forge](https://anaconda.org/conda-forge/basyx-python-sdk):
`conda install -c conda-forge basyx-python-sdk`.

---

## Release Schedule

The Eclipse BaSyx Python SDK team evaluates the main branch bi-monthly. If the accumulated
changes warrant a release, a new version is published to
[PyPI](https://pypi.org/project/basyx-python-sdk/) and
[conda-forge](https://anaconda.org/conda-forge/basyx-python-sdk)
using [semantic versioning](https://semver.org/). If not, the decision is deferred to the
next review. Security fixes may be released at any time.

---

## Contributing

We welcome contributions of all kinds — bug reports, feature requests, documentation
improvements, and code. Please read our [Contribution Guideline](./CONTRIBUTING.md) before
getting started.

### Eclipse Contributor Agreement

To contribute code, you must sign the
[Eclipse Contributor Agreement (ECA)](https://www.eclipse.org/legal/ECA.php).
Create an Eclipse account with the same email address you use for Git commits, then submit
the form at: https://accounts.eclipse.org/user/eca

---

## License

This project is licensed under the terms of the **MIT License**.

SPDX-License-Identifier: MIT

For details on third-party dependencies and their licenses, see the [NOTICE](./NOTICE) file.
