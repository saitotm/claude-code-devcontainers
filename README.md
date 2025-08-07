# Claude Code DevContainer with DOOD Support

[Claude Code](https://claude.ai/code) development containers with Docker-outside-of-Docker (DooD) capability for different programming languages.

## Features

- 🐳 **Docker-outside-of-Docker**: Access host Docker daemon from within the container
- 🧠 **Claude Code Integration**: Pre-configured with Claude Code extension
- 📦 **Language-specific configurations**: Tailored environments for different development needs
- 🔧 **Development tools**: Git, VS Code extensions, and language-specific tooling

## Available Configurations

### Base (Node.js/JavaScript)

- Node.js development environment
- ESLint, Prettier, GitLens extensions
- Zsh shell with bash history persistence

### Rust

- Rust development environment with Cargo
- rust-analyzer extension for IDE support
- Same base features as Node.js configuration

### TLA+

- TLA+ formal specification language environment
- TLC model checker for verification
- PlusCal algorithm language support
- LaTeX integration for PDF generation
- Pre-installed CLI tools:
  - `tlc` - TLA+ model checker
  - `pcal` - PlusCal translator
  - `sany` - TLA+ syntax analyzer
  - `tla2tex` - TLA+ to LaTeX converter

## Usage

1. Clone this repository:

   ```bash
   git clone https://github.com/saitotm/claude-code-devcontainer-for-dood.git
   cd claude-code-devcontainer-for-dood
   ```

2. Choose your development environment. For example:

   - For Node.js/JavaScript: Use `base/.devcontainer/`

3. Copy the desired `.devcontainer` folder to your project root:

   ```bash
   # Example for Node.js projects
   cp -r base/.devcontainer /path/to/your/project/
   ```

4. Open your project in VS Code and select "Reopen in Container"

## Requirements

- Docker Desktop or Docker Engine
- VS Code with Dev Containers extension
