# DOOD Rust DevContainer

Rust development environment with Docker-outside-of-Docker support, optimized for Claude Code.

## Features

- Rust development environment with Cargo and Docker access
- rust-analyzer extension for IDE support
- Zsh shell with bash history persistence
- Access to host Docker daemon from within the container
- Claude Code integration

## Usage

Copy the `.devcontainer` folder to your Rust project root:

```bash
cp -r .devcontainer /path/to/your/rust-project/
```

Then open your project in VS Code and select "Reopen in Container".

## Docker Access

This configuration uses Docker-outside-of-Docker (DOOD), allowing you to run Docker commands inside the container that interact with the host's Docker daemon.