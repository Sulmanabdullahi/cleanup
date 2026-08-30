# cleanup

A macOS shell script that finds and cleans up developer caches and build artifacts to free up disk space.

## What it does

1. **Reports free disk space** before and after cleanup.
2. **Scans known cache locations**, including:
   - System caches (`~/Library/Caches`, `~/.cache`)
   - Node/JS (npm, yarn, pnpm, bun)
   - Python (pip, poetry, pipenv, pyenv, uv)
   - Docker
   - Homebrew
   - Xcode / iOS Simulator
   - Java/JVM (Gradle, Maven)
   - Go, Rust (Cargo), Terraform
3. **Scans your dev folders** (`~/Documents`, `~/dev`, `~/Developer`) for:
   - Python caches: `__pycache__`, `.pytest_cache`, `.mypy_cache`, `.ruff_cache`, `.tox`, `.nox`
   - Node project folders: `node_modules`, `.next`, `dist`, `build`, `.turbo`, `.parcel-cache`, `.vite`
4. **Prompts before deleting anything** — each category (package-manager caches, Python project caches, Node project folders, macOS cache/Trash) requires a `y/n` confirmation.
5. **Prints a summary** of space recovered and the largest folders in your home directory.

## Usage

```bash
chmod +x cleanup
./cleanup
```

The script only deletes files after you explicitly confirm each prompt — nothing is removed automatically.

## Requirements

- macOS
- bash
