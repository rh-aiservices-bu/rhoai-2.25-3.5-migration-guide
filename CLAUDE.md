# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Documentation-only repo: a migration guide for Red Hat OpenShift AI EUS-to-EUS upgrade (2.25.10 to 3.5). No application code, no tests, no linting. All content is Markdown.

## Build Commands

```bash
make build                # Resolve image digests via skopeo + inject + concatenate
SKIP_DIGESTS=1 make build # Skip digest resolution (doc-only changes)
make resolve-digests      # Only fetch latest digests into images.env
make clean                # Remove generated output, staging, and images.env
```

## Architecture

**Source of truth:** 12 numbered Markdown files in `sections/` (01 through 12). Edit only these.

**Generated output:** `output/` directory. Never edit output directly.

**Image digest pipeline:** Section files use `{{PLACEHOLDER}}` tokens for container images (e.g., `{{RHAI_CLI_IMAGE}}`). At build time, `scripts/resolve-digests.sh` fetches latest sha256 digests via `skopeo` (amd64/linux), writes `images.env`, then the Makefile injects resolved values into a `staging/` copy before concatenation.

**Config files:**
- `images.conf` — maps image keys to registry/tag (what to resolve)
- `image-placeholders.conf` — maps image keys to `{{PLACEHOLDER}}` strings in sections (what to replace)
- `images.env` — generated resolved digests (gitignored)

Section ordering is strict — filenames `01-` through `12-` determine document order. See `sections/README.md` for the full index with line ranges and content mapping.

## Workflow

1. Edit files in `sections/`
2. Run `make build` to regenerate (or `SKIP_DIGESTS=1 make build` for doc-only changes)
3. Commit both section changes and updated output

## Adding a New Image

1. Add `KEY image:tag` to `images.conf`
2. Use `{{KEY_IMAGE}}` placeholder in the section file
3. Add `KEY {{KEY_IMAGE}}` to `image-placeholders.conf`

## Key Context

- Guide covers: Model Serving, Workbenches, TrustyAI, OGX (formerly Llama Stack), AI Pipelines, Ray Training Operator, Kubeflow Training Operator
- Related Jira: RHAISTRAT-1519 (Automated Upgrade Validation), RHAISTRAT-1480 (Automated Migration)
- CLI tool: `rhai-cli` (`registry.redhat.io/rhoai/rhoai-cli-rhel9:v3.5`)
- Largest section: `07-ch2-before-modelserving.md` (Model Serving, ~1039 lines)
- Skopeo timeout defaults to 30s; override with `SKOPEO_TIMEOUT=60`
