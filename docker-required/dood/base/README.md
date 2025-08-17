# DOOD Base (Node.js/JavaScript) DevContainer

Node.js development environment with Docker-outside-of-Docker support, optimized for Claude Code.

## Features

- Node.js development environment with Docker access
- ESLint, Prettier, GitLens extensions
- Zsh shell with bash history persistence
- Access to host Docker daemon from within the container
- Claude Code integration

## Usage

Copy the `.devcontainer` folder to your Node.js project root:

```bash
cp -r .devcontainer /path/to/your/node-project/
```

Then open your project in VS Code and select "Reopen in Container".

## Docker Access

This configuration uses Docker-outside-of-Docker (DOOD), allowing you to run Docker commands inside the container that interact with the host's Docker daemon.