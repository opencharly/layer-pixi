# AGENTS.md — layer-pixi

Standalone candy repo for the `pixi` layer — the pinned pixi package-manager
binary. The candy lives in `charly.yml` at the repo root: the `PIXI_VERSION`
var, the `download:`/extract `plan:` steps, the `check:` assertions, the `shell:`
PATH append, and the embedded `skill:` entity projected into the marketplace
corpus as `/charly-languages:pixi`.

Canonical files:

- `charly.yml` — the `pixi:` candy entity and the `pixi-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-languages:pixi` — the owning skill. The pinned release, why
  `releases/latest` must not be reintroduced, and the `pixi.lock` /
  `pixi install --frozen` flip. Load before editing or troubleshooting the
  candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `download:`/`check:`, package sections, service
  declarations). Load before editing any entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- `charly box validate` at the repo root checks the manifest parses and
  validates.
- The candy's `plan:` `check:` steps are the functional evidence: the binary at
  `/usr/local/bin/pixi` and `pixi --version` reporting the pinned release. They
  must stay valid on every distro arm they run on.
- Bump `PIXI_VERSION` and the candy `version:` together; the pinned URL is what
  keeps the same candy version installing the same pixi.

## Modify this repo

- Edit the `pixi:` candy entity AND the `pixi-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a version
  or behaviour change not mirrored in the skill leaves the corpus stale.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
