# MN-Core Development DevContainer

MN-Core program development environment with integrated assembler and emulator.

## Overview

This project provides a devcontainer for developing MN-Core 2 programs. It includes the MN-Core 2 assembler (`assemble3`) and emulator (`gpfn3_package_main`) for developing and testing assembly programs.

## Features

- Pre-configured MN-Core 2 assembler and emulator
- Zsh shell with bash history persistence
- Claude Code integration

## MN-Core 2 Tools

### Assembler

The MN-Core 2 assembler (`assemble3`) converts assembly language programs (described in Chapter 3 of the Software Development Manual) into machine code.

Example usage:

```bash
echo 'lpassa $lm0v $ln0v' > pass.vsm
assemble3 pass.vsm > pass.asm
```

### Emulator

The MN-Core 2 board emulator (`gpfn3_package_main`) allows testing programs without requiring packed instruction streams and supports debug output statements (`d get` - see SDM Section 3.4.3).

Example usage:

```bash
cat << 'EOF' > sample.vsm
lpassa $subpeid $lm0
d get $lm0n0c0b0m0 1
EOF

assemble3 sample.vsm > sample.asm
gpfn3_package_main -i sample.asm -d sample.dmp
cat sample.dmp
```

## Documentation

Software Development Manual (SDM): https://projects.preferred.jp/mn-core/assets/mncore2_dev_manual_ja.pdf

## Usage

Copy the `.devcontainer` folder to your project root:

```bash
cp -r .devcontainer /path/to/your/project/
```

Then open your project in VS Code and select "Reopen in Container".
