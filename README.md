# pixi

The [pixi](https://github.com/prefix-dev/pixi) package manager for OpenCharly images.

The `pixi` candy downloads a **pinned** pixi release binary and extracts it to
`/usr/local/bin/pixi`. pixi is the conda-forge package and environment manager
that backs the `python` / `python-ml` candies and the Fedora/Arch builder
images.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `pixi` |
| Binary | `/usr/local/bin/pixi` |
| Version | pinned `v0.70.2` (`PIXI_VERSION` var) |
| Install | `download:` the musl release tarball for `${BUILD_ARCH}`, extract to `/usr/local/bin` |
| PATH additions | `~/.pixi/bin` |
| Service / port | none |

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
my-dev-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-pixi:v2026.239.1634'
```

Then, inside the built image (or on a dev host):

```bash
pixi --version          # reports the pinned 0.70.2
pixi install            # materialize the environment from pixi.toml
```

The candy's `plan:` asserts the binary at the fixed path and that
`pixi --version` reports the pinned release — a non-functional binary fails the
check.

## Version pinning

`PIXI_VERSION` drives the `releases/download/${PIXI_VERSION}/…` URL. Bump it (and
the candy `version:`) deliberately to adopt a newer pixi; do not revert to an
unpinned `releases/latest` URL, which would make the same candy version install a
different pixi over time.

## Layout

- `charly.yml` — the `pixi:` candy entity (the `PIXI_VERSION` var, the
  `download:`/extract `plan:`, the `check:` assertions, and the `shell:`
  PATH append) and the embedded `pixi-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-languages:pixi`
- Consumers: `/charly-languages:python`, `/charly-languages:python-ml`,
  `/charly-coder:pre-commit`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
