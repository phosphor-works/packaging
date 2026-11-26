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

The repository has no files yet. PlasmaZones still releases from its own `release.yml` and `kwin-rebuild-watch.yml`.
Phase 0 of the Phosphor Split Plan moves them here.

### The plan

The apps build the phosphor libraries statically into their own tree, from the commits that they pin. The
libraries are 0.x with an unstable ABI and use Qt private APIs, so a separate library package would only add a
rebuild of every app on most library releases. Each library repo still has its own semver tags (workspace#40).
Library packages come when a library reaches 1.0 or an outside consumer needs one.

1. Move PlasmaZones' release and rebuild-watch workflows and all of `PlasmaZones/packaging/` here, and delete them
   from PlasmaZones in the same change (workspace#43). The recipes keep bundling the pinned library archives. The
   test: PlasmaZones 3.6.0 published entirely from this repository. The next PlasmaZones releases are 3.x. 4.0 is
   the release with the settings app changes.
2. `graph.yml` names who depends on whom, and the watcher rebuilds the dependents of a KWin release and of each Qt
   minor release (workspace#44). A library release reaches an app through a pin bump in the app.
3. The CI images with the system dependencies installed, for the apps' pull request CI (workspace#42).
4. After step 1, the distribution packages move to the openSUSE Build Service, one format at a time (workspace#54):
   Fedora, openSUSE, Debian, Ubuntu and an Arch binary repository. OBS rebuilds a package when KWin or Qt changes
   in its distribution, which replaces the rebuild watch. The AUR source package and the Nix flake stay as they are.

PlasmaZones moves onto the shared C++ CI in the same phase (workspace#45).

## Layout

There are no files yet, so there is no layout to describe.

## Build and test

There is no build, no CI and no checks yet.

## Rules

### Rules for recipes

- Every recipe repeats each pin of the repo it packages exactly: version, URL and SHA-256. A check here enforces
  that, including the archives of the phosphor-libs commit that a recipe bundles (workspace#46, in `tools/sync --check` of
  the workspace until the recipes move here).
- Formats: rpm (COPR), deb, Arch (a split PKGBUILD for the release, three `-git` PKGBUILDs) and Nix (an overlay).
- The COPR package of PlasmaZones still clones `github.com/fuddlesworth/PlasmaZones`. Fix its clone URL in COPR's
  package settings when the recipes move here.

### When files land

Add the layout, the commands and the CI to this file.
