# CIVL DevContainer

[CIVL](https://vsl.cis.udel.edu/trac/civl/wiki) verification framework environment optimized for Claude Code.

## Features

- CIVL concurrent program verification framework
- Pre-installed CIVL CLI tools:
  - `civl verify` - Verify concurrent C programs
  - `civl run` - Execute CIVL-C programs
  - `civl replay` - Replay error traces
  - `civl show` - Display program transformations
- Support for:
  - MPI program verification
  - OpenMP program verification
  - CUDA program verification
  - Pthread program verification
- Claude Code integration

## Usage

Copy the `.devcontainer` folder to your CIVL project root:

```bash
cp -r .devcontainer /path/to/your/civl-project/
```

Then open your project in VS Code and select "Reopen in Container".

## Version Configuration

The CIVL version can be configured via Docker build arguments in the Dockerfile:

- `CIVL_VERSION`: Major.minor version (default: 1.20)
- `CIVL_REVISION`: Build revision number (default: 5259)

## Example Commands

```bash
# Verify a concurrent C program
civl verify program.c

# Run with specific bounds
civl verify -inputB=10 program.c

# Show transformed program
civl show -showProgram program.c

# Replay an error trace
civl replay -showTransitions program.c
```