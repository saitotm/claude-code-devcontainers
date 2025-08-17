# Claude Code DevContainers

Pre-configured development containers optimized for [Claude Code](https://claude.ai/code), enabling AI-assisted development across various programming languages and environments.

## Features

- 🧠 **Claude Code Integration**: Pre-configured with Claude Code extension
- 📦 **Language-specific configurations**: Tailored environments for different development needs
- 🔧 **Development tools**: Git, VS Code extensions, and language-specific tooling
- 🐳 **Optional Docker support**: DOOD capability available for environments that need it

## Available Configurations

### Basic Environments

#### TLA+
- TLA+ formal specification language environment
- TLC model checker for verification
- PlusCal algorithm language support
- LaTeX integration for PDF generation
- Pre-installed CLI tools:
  - `tlc` - TLA+ model checker
  - `pcal` - PlusCal translator
  - `sany` - TLA+ syntax analyzer
  - `tla2tex` - TLA+ to LaTeX converter

### Docker-Required Environments

#### DOOD Base (Node.js/JavaScript)
- Node.js development environment with Docker access
- ESLint, Prettier, GitLens extensions
- Zsh shell with bash history persistence
- Access to host Docker daemon from within the container

#### DOOD Rust
- Rust development environment with Cargo and Docker access
- rust-analyzer extension for IDE support
- Same base features as Node.js configuration
- Access to host Docker daemon from within the container

## Usage

1. Clone this repository:

   ```bash
   git clone https://github.com/saitotm/claude-code-devcontainers.git
   cd claude-code-devcontainers
   ```

2. Choose your development environment based on your needs:

   - **Basic**: Use configurations from `basic/` for environments that don't require Docker
   - **Docker-required**: Use configurations from `docker-required/` if you need Docker access

3. Copy the desired `.devcontainer` folder to your project root:

   ```bash
   # Example for TLA+ projects (no Docker required)
   cp -r basic/tlaplus/.devcontainer /path/to/your/project/
   
   # Example for Node.js projects with Docker support
   cp -r docker-required/dood/base/.devcontainer /path/to/your/project/
   ```

4. Open your project in VS Code and select "Reopen in Container"

## Requirements

- Docker Desktop or Docker Engine
- VS Code with Dev Containers extension
