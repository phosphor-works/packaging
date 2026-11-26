<!-- SPDX-FileCopyrightText: 2026 Phosphor Works Team -->
<!-- SPDX-License-Identifier: GPL-3.0-or-later -->

<!-- BEGIN shared: tools/sync writes this block from workspace/sync/claude-header.md. Change it there. -->
# packaging

This file has two parts. The rules that every phosphor-works repository follows come first, from the conventions of
the workspace repository, imported below. The sections after them belong to this repository only: what it is, its
layout, how to build and test it, and its own rules, code, skills and agents, and links to the other repositories.
A section here does not repeat a convention. When a section and the conventions disagree, change one of them so
that they agree.

The Claude Code tooling comes from the plugins that `.claude/settings.json` enables and `workspace/tools/setup`
installs. The `phosphor-works` plugin has the command guard, the shared skills (`phosphor-works:build`, `debug`,
`code-audit`, `record-lesson`, `new-repo`), the shared reviewers, and the lesson index, which adds the lessons that
fit each prompt.

@../workspace/claude/conventions.md
<!-- END shared -->

## Overview

The packaging repository of the phosphor-works suite: every recipe, the release workflows, the dependency graph
(`graph.yml`), the rebuild cascades and the CI images. Releases are built here, never through the workspace
superbuild.

### State

PlasmaZones releases from here: its recipes are in `plasmazones/`, and its release and KWin rebuild-watch workflows
are in `.github/workflows/` (workspace#43). The other apps do not release from here yet.

### The plan

The apps build the phosphor libraries statically into their own tree, from the commits that they pin. The
libraries are 0.x with an unstable ABI and use Qt private APIs, so a separate library package would only add a
rebuild of every app on most library releases. Each library repo still has its own semver tags (workspace#40).
Library packages come when a library reaches 1.0 or an outside consumer needs one.

1. Move PlasmaZones' release and rebuild-watch workflows and its release recipes here, and delete them from
   PlasmaZones in the same change (workspace#43). The recipes keep bundling the pinned library archives. The
   test: PlasmaZones 3.6.0 published entirely from this repository. The next PlasmaZones releases are 3.x. 4.0 is
   the release with the settings app changes.
2. `graph.yml` names who depends on whom, and the watcher rebuilds the dependents of a KWin release and of each Qt
   minor release (workspace#44). A library release reaches an app through a pin bump in the app.
3. The CI images with the system dependencies installed, for the apps' pull request CI (workspace#42).

PlasmaZones moves onto the shared C++ CI in the same phase (workspace#45).

## Layout

```
plasmazones/                          — the recipes of PlasmaZones
  arch/                               — PKGBUILD (AUR plasmazones), PKGBUILD.bin, PKGBUILD.git, the pacman hook
  debian/                             — the Debian recipe, built in CI and on OBS
  rpm/                                — plasmazones.spec, built in CI, on OBS and on COPR
  obs/                                — the OBS source service and generate-dsc.sh
  local-install/                      — install.sh and uninstall.sh of the release tarball, and web-install.sh
  generate-changelog.sh               — the Debian and RPM changelogs and the release notes, from CHANGELOG.md
.github/workflows/
  plasmazones-release.yml             — builds and publishes a PlasmaZones tag
  plasmazones-rebuild-watch.yml       — rebuilds the latest release when a distribution moves KWin
  plasmazones-recipes.yml             — checks the recipes against PlasmaZones
  cache-cleanup.yml                   — deletes the compiler caches no release run restores
```

The Nix recipe and modules of PlasmaZones stay in `PlasmaZones/nix/`, beside the flake that imports them. The COPR
package builds from a webhook on PlasmaZones: its `.copr/Makefile` takes `plasmazones/rpm/plasmazones.spec` and
`plasmazones/generate-changelog.sh` from this repository.

### The release workflow

| Trigger | What runs |
| --- | --- |
| `repository_dispatch` of type `plasmazones-release`, with `client_payload.tag` | The release of that tag. PlasmaZones' `release-dispatch.yml` sends it when a `v*` tag is pushed. |
| `workflow_dispatch` with `tag` (and `pkgrel` for a mutable release) | The release of that tag, by hand or from the rebuild watch. |
| `workflow_dispatch` with `warm_cache`, and the nightly schedule | Every package build from PlasmaZones' default branch, to keep the compiler caches fresh. Nothing is published. |

Each job checks out PlasmaZones at the tag and copies `plasmazones/` of this repository to `packaging/` in that tree,
so the recipes run with the same paths as in the source tree of the package. A dispatch from a tag push for a
revision tag (`v<version>-r<N>`) is a no-op: the rebuild watch cuts that tag and starts the run that ships it.

### Secrets

The release secrets are organization secrets of phosphor-works, shared with the PlasmaZones and packaging
repositories, so each value is set once. GitHub never shows a value again: set one from its source with
`gh secret set <NAME> --org phosphor-works --repos PlasmaZones,packaging`.

| Secret | Used by | Access |
| --- | --- | --- |
| `RELEASE_TAG_TOKEN` | packaging, PlasmaZones | Token with Contents and Workflows write on phosphor-works/PlasmaZones, and Contents write on phosphor-works/packaging. The release job makes the GitHub release with it, the rebuild watch cuts revision tags with it, and PlasmaZones' release dispatch starts the release with it. |
| `AUR_SSH_PRIVATE_KEY` | packaging | The private SSH key (not the `.pub` file) of the AUR account that maintains `plasmazones` and `plasmazones-bin`. |
| `OBS_USERNAME`, `OBS_PASSWORD` | packaging | The build.opensuse.org account of `home:fuddlesworth`. |

## Build and test

Nothing here builds on its own. The release workflow builds the packages in CI. To build one package by hand, copy
`plasmazones/` to `packaging/` in a PlasmaZones checkout and run the steps of its job in `plasmazones-release.yml`.

The checks of the recipes are PlasmaZones' `dep5`, `dep-pins` and `recipe-prose` rules. They read this checkout
through `PLASMAZONES_PACKAGING_DIR`, or as `../packaging` beside a PlasmaZones checkout:

```bash
PLASMAZONES_PACKAGING_DIR=$PWD python3 ../PlasmaZones/scripts/check-conventions.py --rules dep5,dep-pins,recipe-prose
```

`plasmazones-recipes.yml` runs them on every pull request and push, against the PlasmaZones branch of the same name
when there is one and against `main` otherwise. PlasmaZones' CI runs them against this repository the same way.
`workspace/tools/sync --check` also checks each archive that a recipe bundles against the pins of the app.

Do not run `plasmazones-release.yml` or `plasmazones-rebuild-watch.yml` to test a change: both publish. Use
`warm_cache` to build every package and publish nothing, and `dry_run` of the rebuild watch to see what it would do.

## Rules

### Rules for recipes

- Every recipe repeats each pin of the repo it packages exactly: version, URL and SHA-256. `plasmazones-recipes.yml`
  and `tools/sync --check` of the workspace enforce that, including the archives of the phosphor-libs commit that a
  recipe bundles (workspace#46).
- A recipe change that needs a change in the app (a pin bump, a new file in `debian/copyright`) is a change set: the
  same branch name here and in the app.
- Formats: rpm (COPR), deb, Arch (a split PKGBUILD for the release, three `-git` PKGBUILDs) and Nix (an overlay).
- The COPR package of PlasmaZones still clones `github.com/fuddlesworth/PlasmaZones`. Fix its clone URL in COPR's
  package settings to `https://github.com/phosphor-works/PlasmaZones`.
