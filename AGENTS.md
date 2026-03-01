# AI Agent Guidelines

## Project Overview

LaTeX cheatsheet for the Atom text editor (`atom_cheatsheet.tex`),
compiled to PDF/JPG/PNG/SVG via Docker. No application code -- the
primary artifacts are the `.tex` source and supporting shell/YAML files.

## Build Commands

```bash
# Full build (PDF + images) via Docker -- requires Docker running
./run.sh

# Build only PDF
./run.sh pdf

# Build specific formats
./run.sh jpg pdf png svg mostlyclean

# Use custom Docker image
./run.sh -i custom/texlive pdf

# Verbose build
./run.sh --verbose pdf

# Direct Make (requires local TeX Live installation)
make             # Default: build PDF
make pdf         # PDF only
make png         # PNG only
make lint        # Run chktex LaTeX linter
make pretty      # Run latexindent formatter
make clean       # Remove all generated files
make mostlyclean # Remove intermediate files only
```

## Lint and Validation Commands

```bash
# Shell scripts
shellcheck --exclude=SC2317 run.sh
shfmt --case-indent --indent 2 --space-redirects --diff run.sh

# Markdown
rumdl file.md

# Link checking
lychee --accept 200,429 file.md

# JSON (supports comments)
jsonlint --comments file.json

# LaTeX
chktex atom_cheatsheet.tex

# GitHub Actions workflows
actionlint

# Makefile
checkmake Makefile
```

There is no test suite. The LaTeX compilation itself serves as the
validation test. CI runs MegaLinter (`.mega-linter.yml`) which
orchestrates all linting tools listed above.

## Code Style

### General Formatting

- Two spaces for indentation everywhere (YAML, JSON, shell, Markdown)
- No tabs
- Files must end with a newline
- Wrap Markdown lines at 72 characters
- `CHANGELOG.md` is auto-generated -- never edit it manually

### Shell Scripts

- Start with `#!/usr/bin/env bash` and `set -euo pipefail`
- Uppercase for all variables: `${MY_VARIABLE}`
- Use `${var}` brace syntax, not `$var`
- Format with `shfmt --case-indent --indent 2 --space-redirects`
- Must pass `shellcheck` (SC2317 excluded)
- Redirect stderr: `command > /dev/null 2>&1`
- Use `[[ ]]` for conditionals, not `[ ]`

### LaTeX

- Linter: `chktex` (warnings 1, 8, 44 disabled via `.chktexrc`)
- Suppress specific warnings inline: `% chktex <number>`
- Use `\keys{}` macro from `menukeys` for keyboard shortcuts
- Use `\textbf{}` to highlight mnemonic letters

### YAML (GitHub Actions Workflows)

- Start files with `---`
- Use `# keep-sorted start` / `# keep-sorted end` blocks for
  alphabetically sorted sections
- Descriptive step names
- Always set `permissions: read-all` (least privilege)
- Pin all actions to full SHA commits, never use tags
- Set `timeout-minutes` on every job (5-10 minutes)
- Validate changes with `actionlint` before committing

### Markdown

- Use proper heading hierarchy (no skipped levels)
- Include language identifiers in code fences
- Shell code blocks (`bash`/`shell`/`sh`) are extracted and validated
  by ShellCheck and shfmt during CI
- Prefer code fences over inline code for multi-line examples

### JSON

- Must pass `jsonlint --comments`
- `.devcontainer/devcontainer.json` is excluded from linting

## Security Scanning

CI runs these scanners -- avoid introducing violations:

- **Checkov**: IaC scanner (skip `CKV_GHA_7`)
- **DevSkim**: Security patterns (ignore DS162092, DS137138)
- **KICS**: Fails on HIGH severity only
- **Trivy**: HIGH/CRITICAL only, ignores unfixed vulnerabilities

## Version Control

### Commit Messages

Conventional commit format: `<type>: <description>`

- Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `style`,
  `perf`, `ci`, `build`, `revert`
- Subject: imperative mood, lowercase, no period, max 72 characters
- Body: wrap at 72 characters, explain what and why
- Reference issues: `Fixes #123`, `Closes #456`, `Resolves #789`

```text
feat: add automated dependency updates

- Implement Dependabot configuration
- Configure weekly security updates

Resolves: #123
```

### Branching

Follow [Conventional Branch](https://conventional-branch.github.io/)
format: `<type>/<description>`

- `feature/` or `feat/`: new features
- `bugfix/` or `fix/`: bug fixes
- `hotfix/`: urgent fixes
- `release/`: releases (e.g., `release/v1.2.0`)
- `chore/`: non-code tasks

Use lowercase, hyphens, no consecutive/leading/trailing hyphens.
Include issue number when applicable: `feature/issue-42-add-shortcuts`

### Pull Requests

- Always create as **draft** initially
- Title must follow conventional commit format
- Include clear description of changes and motivation
- Link related issues with `Fixes`, `Closes`, or `Resolves`

## Quality Checklist

Before committing, verify:

- [ ] `shellcheck` passes on all shell scripts
- [ ] `shfmt --diff` shows no formatting changes
- [ ] `rumdl` passes on all Markdown files
- [ ] `actionlint` passes if workflow files were modified
- [ ] `chktex` passes on LaTeX files
- [ ] Lines wrapped at 72 characters in Markdown
- [ ] Commit message follows conventional format
