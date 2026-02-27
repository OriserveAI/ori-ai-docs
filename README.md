# Oriserve AI Services Documentation

This repo contains the Sphinx-based documentation for the Oriserve AI Services (STT and TTS).

## Project Structure

```
speech-docs/
├── source/               # All documentation source files (edit here)
│   ├── conf.py           # Sphinx configuration
│   ├── index.rst         # Root table of contents
│   ├── stt/              # Speech-to-Text docs
│   │   ├── index.rst
│   │   ├── quickstart.rst
│   │   ├── api_reference.rst
│   │   ├── python_client.rst
│   │   ├── examples.rst
│   │   └── troubleshooting.rst
│   └── tts/              # Text-to-Speech docs
│       ├── index.rst
│       ├── quickstart.rst
│       └── api/
│           ├── index.rst
│           ├── tts.rst
│           └── voice.rst
├── docs/                 # Built HTML output (do not edit manually)
├── Makefile
└── requirements.txt
```

## Setup

Install the required Python packages:

```bash
pip install -r requirements.txt
```

## Making Changes

All source files are under `source/`. Edit the `.rst` files directly.

- To add a new page, create a `.rst` file in the appropriate subdirectory (`stt/` or `tts/`), then add it to the relevant `toctree` in `source/index.rst` or the section's `index.rst`.
- The theme and Sphinx settings are in `source/conf.py`.

## Building the Docs

Run from the repo root:

```bash
make html
```

The built site is output to `docs/`. The `docs/.nojekyll` file is created automatically so GitHub Pages serves it correctly.

To remove the previous build before rebuilding:

```bash
make clean && make html
```

## Local Preview

Open the built HTML directly in a browser:

```bash
open docs/index.html
```

Or serve it with Python's built-in HTTP server to get proper relative-path behavior:

```bash
python3 -m http.server 8000 --directory docs
```

Then visit `http://localhost:8000` in your browser.
