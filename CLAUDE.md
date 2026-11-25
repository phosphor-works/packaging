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

1. Library recipes first: phosphor-libs, phosphor-engine-libs, phosphor-shell-libs, phosphor-components and
   phosphor-surface-packs, each as its own package once each library repo has tags.
2. Move PlasmaZones' release and rebuild-watch workflows here. The test: a PlasmaZones patch release published
   entirely from this repository.
3. `graph.yml` names who depends on whom, and the watcher rebuilds the dependents of a library release, of a KWin
   release and of each Qt minor release (the libraries use Qt private APIs).
4. The CI images with the libraries installed, for the apps' pull request CI.

## Layout

There are no files yet, so there is no layout to describe.

## Build and test

There is no build, no CI and no checks yet.

## Rules

### Rules for recipes

- Every recipe repeats each pin of the repo it packages exactly: version, URL and SHA-256. A check here enforces
  that, the way PlasmaZones' dep-pins rule does today.
- Formats: rpm (COPR), deb, Arch (a split PKGBUILD for the release, three `-git` PKGBUILDs) and Nix (an overlay).
- The COPR package of PlasmaZones still clones `github.com/fuddlesworth/PlasmaZones`. Fix its clone URL in COPR's
  package settings when the recipes move here.

### When files land

Add the layout, the commands and the CI to this file.
