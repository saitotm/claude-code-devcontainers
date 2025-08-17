# TLA+ DevContainer

TLA+ formal specification language environment optimized for Claude Code.

## Features

- TLA+ formal specification language environment
- TLC model checker for verification
- PlusCal algorithm language support
- LaTeX integration for PDF generation
- Pre-installed CLI tools:
  - `tlc` - TLA+ model checker
  - `pcal` - PlusCal translator
  - `sany` - TLA+ syntax analyzer
  - `tla2tex` - TLA+ to LaTeX converter
- CLAUDE.md configuration for Claude Code assistant

## Usage

Copy both the `.devcontainer` folder and `CLAUDE.md` file to your TLA+ project root:

```bash
cp -r .devcontainer CLAUDE.md /path/to/your/tla-project/
```

Then open your project in VS Code and select "Reopen in Container".