#!/usr/bin/env bash
set -euo pipefail

echo "======================================"
echo " Find & Clean Developer Caches on macOS"
echo "======================================"
echo

bytes_available() {
  df -k / | awk 'NR==2 {print $4 * 1024}'
}

human_bytes() {
  local bytes="$1"
  awk -v b="$bytes" '
    function human(x) {
      s="B KiB MiB GiB TiB"
      split(s, a, " ")
      i=1
      while (x >= 1024 && i < 5) {
        x /= 1024
        i++
      }
      return sprintf("%.2f %s", x, a[i])
    }
    BEGIN { print human(b) }
  '
}

START_FREE_BYTES="$(bytes_available)"

echo "Disk before cleanup:"
df -h /
echo
echo "Free before: $(human_bytes "$START_FREE_BYTES")"
echo

HOME_DIR="$HOME"

CACHE_PATHS=(
  "$HOME_DIR/Library/Caches"
  "$HOME_DIR/.cache"

  # Node / JS
  "$HOME_DIR/.npm"
  "$HOME_DIR/.yarn"
  "$HOME_DIR/Library/Caches/Yarn"
  "$HOME_DIR/Library/Caches/npm"
  "$HOME_DIR/Library/pnpm"
  "$HOME_DIR/.pnpm-store"
  "$HOME_DIR/Library/Caches/pnpm"
  "$HOME_DIR/.bun"
  "$HOME_DIR/Library/Caches/Bun"

  # Python
  "$HOME_DIR/Library/Caches/pip"
  "$HOME_DIR/.cache/pip"
  "$HOME_DIR/.cache/pypoetry"
  "$HOME_DIR/Library/Caches/pypoetry"
  "$HOME_DIR/.cache/pipenv"
  "$HOME_DIR/.local/share/virtualenvs"
  "$HOME_DIR/.pyenv/cache"
  "$HOME_DIR/.uv"
  "$HOME_DIR/Library/Caches/uv"

  # Docker
  "$HOME_DIR/Library/Containers/com.docker.docker"
  "$HOME_DIR/Library/Group Containers/group.com.docker"

  # Homebrew
  "$(brew --cache 2>/dev/null || true)"

  # Xcode / Apple dev
  "$HOME_DIR/Library/Developer/Xcode/DerivedData"
  "$HOME_DIR/Library/Developer/CoreSimulator/Caches"
  "$HOME_DIR/Library/Developer/CoreSimulator/Devices"

  # Java / JVM
  "$HOME_DIR/.gradle/caches"
  "$HOME_DIR/.m2/repository"

  # Go
  "$HOME_DIR/Library/Caches/go-build"
  "$HOME_DIR/go/pkg/mod"

  # Rust
  "$HOME_DIR/.cargo/registry"
  "$HOME_DIR/.cargo/git"

  # Terraform
  "$HOME_DIR/.terraform.d/plugin-cache"
  "$HOME_DIR/Library/Caches/com.apple.dt.Xcode"
)

echo "Scanning known cache locations..."
echo

FOUND_PATHS=()

for path in "${CACHE_PATHS[@]}"; do
  if [[ -n "$path" && -e "$path" ]]; then
    FOUND_PATHS+=("$path")
  fi
done

if [[ ${#FOUND_PATHS[@]} -eq 0 ]]; then
  echo "No known cache paths found."
else
  printf "%-12s %s\n" "SIZE" "PATH"
  printf "%-12s %s\n" "----" "----"

  for path in "${FOUND_PATHS[@]}"; do
    du -sh "$path" 2>/dev/null || true
  done | sort -h
fi

echo
echo "Finding Python project caches..."
echo

DEV_DIRS=(
  "$HOME_DIR/Documents"
  "$HOME_DIR/documents"
  "$HOME_DIR/dev"
  "$HOME_DIR/Developer"
)

for dir in "${DEV_DIRS[@]}"; do
  if [[ -d "$dir" ]]; then
    echo
    echo "Scanning: $dir"
    find "$dir" \( \
      -name "__pycache__" -o \
      -name ".pytest_cache" -o \
      -name ".mypy_cache" -o \
      -name ".ruff_cache" -o \
      -name ".tox" -o \
      -name ".nox" \
    \) -type d -prune -print 2>/dev/null | while read -r cache; do
      du -sh "$cache" 2>/dev/null || true
    done | sort -h
  fi
done

echo
echo "Finding Node project folders..."
echo

for dir in "${DEV_DIRS[@]}"; do
  if [[ -d "$dir" ]]; then
    echo
    echo "Scanning: $dir"
    find "$dir" \( \
      -name "node_modules" -o \
      -name ".next" -o \
      -name "dist" -o \
      -name "build" -o \
      -name ".turbo" -o \
      -name ".parcel-cache" -o \
      -name ".vite" \
    \) -type d -prune -print 2>/dev/null | while read -r cache; do
      du -sh "$cache" 2>/dev/null || true
    done | sort -h
  fi
done

echo
read -p "Clean safe package-manager caches? npm/yarn/pnpm/pip/poetry/uv/brew/docker build cache (y/n): " CLEAN_SAFE

if [[ "$CLEAN_SAFE" =~ ^[Yy]$ ]]; then
  echo
  echo "Cleaning package-manager caches..."

  if command -v npm >/dev/null 2>&1; then
    npm cache clean --force || true
  fi

  if command -v yarn >/dev/null 2>&1; then
    yarn cache clean || true
  fi

  if command -v pnpm >/dev/null 2>&1; then
    pnpm store prune || true
  fi

  if command -v pip >/dev/null 2>&1; then
    pip cache purge || true
  fi

  if command -v pip3 >/dev/null 2>&1; then
    pip3 cache purge || true
  fi

  if command -v poetry >/dev/null 2>&1; then
    poetry cache clear --all pypi || true
  fi

  if command -v uv >/dev/null 2>&1; then
    uv cache clean || true
  fi

  if command -v brew >/dev/null 2>&1; then
    brew cleanup -s || true
  fi

  if command -v docker >/dev/null 2>&1; then
    docker builder prune -af || true
    docker system prune -af || true
  fi
fi

echo
read -p "Delete Python project caches? __pycache__, .pytest_cache, .mypy_cache, .ruff_cache, .tox, .nox (y/n): " CLEAN_PY_PROJECT

if [[ "$CLEAN_PY_PROJECT" =~ ^[Yy]$ ]]; then
  for dir in "${DEV_DIRS[@]}"; do
    if [[ -d "$dir" ]]; then
      find "$dir" \( \
        -name "__pycache__" -o \
        -name ".pytest_cache" -o \
        -name ".mypy_cache" -o \
        -name ".ruff_cache" -o \
        -name ".tox" -o \
        -name ".nox" \
      \) -type d -prune -exec rm -rf '{}' + 2>/dev/null || true
    fi
  done
fi

echo
read -p "Delete Node project folders? node_modules, .next, dist, build, .turbo, .parcel-cache, .vite (y/n): " CLEAN_NODE_PROJECT

if [[ "$CLEAN_NODE_PROJECT" =~ ^[Yy]$ ]]; then
  for dir in "${DEV_DIRS[@]}"; do
    if [[ -d "$dir" ]]; then
      find "$dir" \( \
        -name "node_modules" -o \
        -name ".next" -o \
        -name "dist" -o \
        -name "build" -o \
        -name ".turbo" -o \
        -name ".parcel-cache" -o \
        -name ".vite" \
      \) -type d -prune -exec rm -rf '{}' + 2>/dev/null || true
    fi
  done
fi

echo
read -p "Clean macOS user cache and Trash? (y/n): " CLEAN_MAC

if [[ "$CLEAN_MAC" =~ ^[Yy]$ ]]; then
  rm -rf "$HOME_DIR/Library/Caches/"* 2>/dev/null || true
  rm -rf "$HOME_DIR/.Trash/"* 2>/dev/null || true
fi

echo
END_FREE_BYTES="$(bytes_available)"
RECOVERED_BYTES="$((END_FREE_BYTES - START_FREE_BYTES))"

echo "Disk after cleanup:"
df -h /
echo

echo "======================================"
echo " Cleanup Summary"
echo "======================================"
echo "Free before: $(human_bytes "$START_FREE_BYTES")"
echo "Free after:  $(human_bytes "$END_FREE_BYTES")"

if [[ "$RECOVERED_BYTES" -gt 0 ]]; then
  echo "Recovered:   $(human_bytes "$RECOVERED_BYTES")"
else
  echo "Recovered:   0 B or space has not been reclaimed yet"
fi

echo
echo "Top largest folders in home:"
du -sh "$HOME_DIR"/* 2>/dev/null | sort -h | tail -20

echo
echo "Done."
