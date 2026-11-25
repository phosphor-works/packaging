<!-- SPDX-FileCopyrightText: 2026 Phosphor Works Team -->
<!-- SPDX-License-Identifier: GPL-3.0-or-later -->

# packaging

The packaging repository of the phosphor-works suite: every recipe, the release workflows, the dependency graph
(`graph.yml`), the rebuild cascades and the CI images. Releases are built here, never through the workspace
superbuild.

The rules every phosphor-works repository follows come from the workspace repository, imported below. This file adds
packaging's own. The shared Claude Code tooling (the command guard, `phosphor-works:build`, `phosphor-works:code-audit`,
the shared reviewers, the lesson index) comes from the `phosphor-works` plugin, which `.claude/settings.json` enables
and `workspace/tools/setup` installs.

@../workspace/claude/conventions.md

## State

The repository has no files yet. PlasmaZones still releases from its own `release.yml` and `kwin-rebuild-watch.yml`.
Phase 0 of the Phosphor Split Plan moves them here.

## The plan

1. Library recipes first: phosphor-libs, phosphor-engine-libs, phosphor-shell-libs, phosphor-components and
   phosphor-surface-packs, each as its own package once each library repo has tags.
2. Move PlasmaZones' release and rebuild-watch workflows here. The test: a PlasmaZones patch release published
   entirely from this repository.
3. `graph.yml` names who depends on whom, and the watcher rebuilds the dependents of a library release, of a KWin
   release and of each Qt minor release (the libraries use Qt private APIs).
4. The CI images with the libraries installed, for the apps' pull request CI.

## Rules for recipes

- Every recipe repeats each pin of the repo it packages exactly: version, URL and SHA-256. A check here enforces
  that, the way PlasmaZones' dep-pins rule does today.
- Formats: rpm (COPR), deb, Arch (a split PKGBUILD for the release, three `-git` PKGBUILDs) and Nix (an overlay).
- The COPR package of PlasmaZones still clones `github.com/fuddlesworth/PlasmaZones`. Fix its clone URL in COPR's
  package settings when the recipes move here.

When files land, add the layout, the commands and the CI to this file.
