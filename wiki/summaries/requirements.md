---
doc_type: short
full_text: sources/requirements.md
---

# Requirements: Python Environment Dependencies

## Overview
This file specifies the pinned Python package dependencies for a Jupyter/IPython-based development environment. All packages are version-locked for reproducibility.

## Core Components

### Jupyter & IPython Stack
- **ipython==9.10.0** — Interactive Python shell and kernel
- **ipykernel==7.2.0** — IPython kernel for Jupyter
- **jupyter-client==8.8.0** — Jupyter messaging protocol client
- **jupyter-core==5.9.1** — Core Jupyter functionality
- **ipython-pygments-lexers==1.1.1** — Syntax highlighting for IPython

### Communication & Messaging
- **pyzmq==27.1.0** — ZeroMQ bindings for inter-process messaging
- **tornado==6.5.4** — Async networking framework used by Jupyter
- **comm==0.2.3** — Jupyter comm protocol implementation

### Interactive Shell & Display
- **prompt-toolkit==3.0.52** — Advanced interactive CLI toolkit
- **pygments==2.19.2** — Syntax highlighting library
- **matplotlib-inline==0.2.1** — Inline matplotlib rendering in notebooks
- **wcwidth==0.6.0** — Terminal character width utilities

### Code Intelligence
- **jedi==0.19.2** — Python autocompletion and static analysis
- **parso==0.8.6** — Python parser used by Jedi
- **asttokens==3.0.1** — AST token mapping
- **executing==2.2.1** — Runtime code execution context
- **pure-eval==0.2.3** — Safe expression evaluation
- **stack-data==0.6.3** — Stack frame data extraction

### Debugging & Development
- **debugpy==1.8.20** — Python debugger (used by VS Code and Jupyter)
- **psutil==7.2.2** — System and process utilities

### Utilities & Infrastructure
- **traitlets==5.14.3** — Configuration and traits system for Jupyter
- **packaging==26.0** — Package version parsing utilities
- **platformdirs==4.5.1** — Cross-platform directory paths
- **nest-asyncio==1.6.0** — Allows nested asyncio event loops
- **python-dateutil==2.9.0.post0** — Date/time parsing utilities
- **six==1.17.0** — Python 2/3 compatibility layer
- **decorator==5.2.1** — Function decorator utilities
- **pip==26.0.1** — Python package installer

### Platform-Specific
- **appnope==0.1.4** — macOS App Nap suppression for background processes
- **pexpect==4.9.0** — Subprocess control (Unix)
- **ptyprocess==0.7.0** — Pseudo-terminal process spawning

## Environment Profile
This is a **Jupyter notebook / interactive Python** environment. It does not include data science libraries (numpy, pandas, etc.) directly, suggesting it may be a base kernel environment on top of which other packages are installed separately. See [[concepts/python-environment-management]] for related dependency management practices used across the project.

## Related Pages
- [[concepts/python-environment-management]]
- [[concepts/python-project-structure]]
- [[concepts/local-first-architecture]]

## Related Concepts
- [[concepts/uv-workspace-layout]]
