---
name: pixi
description: 'Pixi package manager binary with environment and PATH setup.

  Use when working with pixi, conda-forge packages, or Python environment management.'
---

# pixi -- Pixi package manager

## Candy Properties

| Property | Value |
|----------|-------|
| Install files | `charly.yml`, `task:` |

## Environment Variables

| Variable | Value | Note |
|----------|-------|------|
| `PIXI_CACHE_DIR` | `~/.cache/pixi` | the package cache: `pkgs/` (one dir per package) + `uv-cache/`; a non-frozen solve adds `repodata/` |
| `RATTLER_CACHE_DIR` | *(not set)* | measured on pixi 0.70.2 AND 0.84.2: a `--frozen` install **never creates** this root, so the build stage no longer declares it |

PATH additions: `~/.pixi/bin`, `~/.pixi/envs/default/bin`

## Usage

```yaml
# charly.yml
my-image:
  candy:
    - pixi
```

## Used In Boxes

- `/charly-distros:fedora-builder` (direct)
- `/charly-distros:arch-builder` (direct)
- Transitive dependency via `python` / `supervisord` in most service boxes

## Related Candies

- `/charly-languages:python` -- depends on pixi for Python 3.13 installation
- `/charly-coder:pre-commit` -- uses pixi for pre-commit installation

## Pinned pixi version (`var: PIXI_VERSION`)

The `pixi` candy installs a **pinned** pixi release — `var: PIXI_VERSION` drives the
`releases/download/${PIXI_VERSION}/pixi-${BUILD_ARCH}-unknown-linux-musl.tar.gz`
download. Bump `PIXI_VERSION` (and the candy `version:`) deliberately to adopt a
newer pixi; do NOT revert to an unpinned `releases/latest` URL, which makes the
same candy version install a different pixi over time (non-reproducible builds,
against charly's content-hash/CalVer model). Candy `pixi.toml` files use the
modern `[workspace]` table (the deprecated `[project]` form emits a warning under
pixi ≥ 0.70 and is being removed upstream).

Current pin: **v0.84.2**. Measured on it: `--frozen` fetches **no repodata** (the lock
is used, nothing is solved), and the package cache lands in `PIXI_CACHE_DIR` (`pkgs`,
`uv-cache`). The build stage mounts that directory as a BuildKit cache
(`--mount=type=cache,dst=/tmp/pixi-cache,uid=1000,gid=1000`); a controlled build that
added one new package added it to `pkgs/` and a second build re-read it, and the real
daemon held 43 package dirs / 219 MB across builds.

## Committed `pixi.lock` → `pixi install --frozen`

Every pixi candy ships a committed `pixi.lock` next to its `pixi.toml`. When a
lock is present, generation auto-flips the build stage from `pixi install` (a
full SAT solve over the conda + PyPI indexes on every cache miss) to
`pixi install --frozen` (install straight from the lock — no solve,
deterministic, better cache reuse). The flip is automatic via `HasPixiLock`
detection (`charly/layers.go`) → the `pixi.toml+lock` install command in
the embedded build vocabulary (`charly/charly.yml`); no per-candy config.

**Regenerate the lock whenever you change `pixi.toml`** — `--frozen` fails the
build loudly if the lock is stale (no silent skew). Generate with the builder's
own pixi so the lock format matches what installs it.

### `--frozen` resolves a PLATFORM, and the lock must carry it

`pixi install --frozen` does not solve; it installs the platform the **manifest**
names. A `[system-requirements] libc = 2.39` table changes that platform to
`linux-64-glibc-2-39`, so a lock resolved for a bare `linux-64` then fails:

```
Error: the lock file has no entry for platform 'linux-64-glibc-2-39' in environment 'default'
help: Run `pixi lock` to resolve it for this platform.
```

That is a real breakage observed on pixi 0.84.2 (the openwebui image build), not a
hypothetical. The rule:

- The `[system-requirements]` table is **DEPRECATED** in 0.84.2 (it prints a warning
  pointing at virtual packages on `platforms`) but is still accepted.
- It is **not required**: measured on 0.84.2, a manifest with `platforms = ["linux-64"]`
  and **no** `[system-requirements]` locks and installs `--frozen` cleanly.
- The build stage appends the section when the manifest lacks it (so the resolved
  packages stay compatible with the base's glibc). If a candy therefore declares it
  itself — making the append a no-op — the lock **must** be resolved for
  `linux-64-glibc-2-39`:

```bash
# in the candy dir, with the SAME pixi version the builder installs (0.84.2)
PIXI_HOME=$PWD/.pixi-tmp pixi lock      # writes pixi.lock for the manifest's platform
grep -m1 'name: linux-64' pixi.lock     # must name the platform the manifest resolves
```

A lock regenerated this way carries `linux-64-glibc-2-39` and `--frozen` matches.
A candy that does NOT declare the section keeps a bare `linux-64` lock, which is
correct only while the build stage does not append — so **declare the section and
lock for it, or leave both alone; never declare it and keep a bare-platform lock.**

## When to Use This Skill

Use when the user asks about:

- Pixi package manager setup
- Conda-forge package installation
- Python environment base candy
- The `pixi` candy or `PIXI_CACHE_DIR`

## Related

- `/charly-image:layer` — candy authoring reference (`charly.yml` schema, task verbs, service declarations)
- `/charly-check:check` — declarative testing (`check:` block, `charly check box`, `charly check live`)
