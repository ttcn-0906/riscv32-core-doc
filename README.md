# riscv32-core-doc

## Overview

This repository contains the documentation for the RISC-V 32-bit core. The documentation is built using Sphinx, which supports both reStructuredText and Markdown formats.

## Getting Started

### Installation

Navigate to the docs directory and install the required Python packages from the `requirements.txt` file.

```
cd docs
pip install -r requirements.txt
```

### Building the Documentation

To compile the documentation into a viewable HTML format, use the `make` command. The generated files will be placed in the `docs/build` directory.

```
cd docs
make html
```

## Usage Guide

### Adding New Content

Documentation pages can be created using either **reStructuredText** (`.rst`) or **Markdown** (`.md`) file formats.
- Place all new documentation files within the docs/source directory.
- To include a new page in the navigation structure, add a reference to it in the appropriate `*.rst` index file's toctree. This links the page into the documentation's table of contents.
