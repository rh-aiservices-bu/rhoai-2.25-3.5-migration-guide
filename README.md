# RHOAI 2.25 to 3.5 Migration Guide

> [!WARNING]
> # ⚠️ DRAFT — WORK IN PROGRESS — NOT FOR CUSTOMER USE
>
> **This document is an unfinished internal draft. Do not share it with customers, and do not use it to perform a migration.**
>
> **OpenShift AI 3.5 has not been released.** The current release of Red Hat OpenShift AI is **3.4**. Everything in this repository describes a future version and is subject to change without notice.
>
> **For a real 2.25 migration, use the official Red Hat documentation instead:**
> **[Migrating from Red Hat OpenShift AI 2.25 (access.redhat.com)](https://access.redhat.com/articles/7133758?extIdCarryOver=true&sc_cid=RHCTG0180000382538)**

Migration guide for Red Hat OpenShift AI EUS-to-EUS upgrade (2.25.10 to 3.5).

This guide provides step-by-step instructions for cluster administrators performing an in-place or side-by-side migration from OpenShift AI 2.25.10 (and later) to 3.5, covering all components: Model Serving, Workbenches, TrustyAI, OGX (formerly Llama Stack), AI Pipelines, Ray Training Operator, and Kubeflow Training Operator.

## Directory structure

```
sections/               12 markdown files, one per chapter/topic (edit here)
output/                 Generated concatenated document (do not edit directly)
images.conf             Image keys mapped to registry/tag (source of truth)
image-placeholders.conf Maps image keys to {{PLACEHOLDER}} strings in sections
scripts/                Build helper scripts
  resolve-digests.sh    Fetches latest sha256 digests via skopeo
images.env              Generated digest file (gitignored)
staging/                Temp dir for placeholder injection (gitignored)
Makefile                Orchestrates digest resolution, injection, and concatenation
```

See [sections/README.md](sections/README.md) for the full section index.

## Build the document

```bash
make build
```

This resolves the latest container image digests via `skopeo`, injects them into section placeholders, and concatenates all section files into:

```
output/Migrate from OpenShift AI 2.25 to 3.5.md
```

For documentation-only changes (no image updates needed), skip digest resolution:

```bash
SKIP_DIGESTS=1 make build
```

This reuses the existing `images.env` file instead of calling `skopeo`.

To remove generated output, staging, site, and images.env:

```bash
make clean
```

### Render the HTML page

```bash
make html
```

This runs the full build, then renders `site/index.html` via `pandoc` (plus a copy
of the Markdown and a `.nojekyll` marker). `site/` is gitignored.

The `.github/workflows/pages.yml` workflow runs the same `make html` on every push
to `main` (and on demand via **Actions → Publish migration guide → Run workflow**),
then deploys `site/` to GitHub Pages. It requires **Settings → Pages → Source** to be
set to **GitHub Actions**.

Rendering uses pandoc's `markdown` reader rather than `gfm`, because the guide's
headings carry explicit `{#id}` attributes that its hand-authored table of contents
links to; the `gfm` reader does not honour them.

### Image digest management

Container image references in section files use `{{PLACEHOLDER}}` tokens (e.g., `{{RHAI_CLI_IMAGE}}`). At build time:

1. `scripts/resolve-digests.sh` reads `images.conf`, calls `skopeo inspect --override-arch amd64 --override-os linux` for each image, and writes resolved `image@sha256:digest` values to `images.env`.
2. The Makefile copies sections to `staging/`, replaces placeholders using `image-placeholders.conf` + `images.env`, then concatenates.

If `skopeo` times out (default 30s, configurable via `SKOPEO_TIMEOUT`), the script falls back to the previous digest from `images.env`.

To add a new image:
1. Add a `KEY image:tag` line to `images.conf`
2. Use `{{KEY_IMAGE}}` in the relevant section file
3. Add `KEY {{KEY_IMAGE}}` to `image-placeholders.conf`

## Contributing

1. Edit files in `sections/` (not the generated output).
2. Run `make build` to regenerate (or `SKIP_DIGESTS=1 make build` for doc-only changes).
3. Review the output in `output/`.
4. Commit both the section changes and the updated output.

Section files are numbered `01-` through `12-` and must be kept in order. The Makefile concatenates them sequentially to produce the final document.

## Related

- **RHAISTRAT-1519** - Automated Upgrade Validation feature
- **RHAISTRAT-1480** - Automated Migration: RHOAI 2.25 to 3.5 (EUS) outcome
- **rhai-cli** - Migration assessment and action CLI (`registry.redhat.io/rhoai/rhai-cli-rhel9:v3.5`)
