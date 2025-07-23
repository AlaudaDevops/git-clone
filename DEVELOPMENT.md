# Git alauda Branch Development Guide

## Background

Previously, git was used as a general-purpose CLI in multiple plugins, each needing to fix git's own vulnerabilities independently.

To avoid duplicate work, we forked the current repository from [git](https://github.com/tektoncd-catalog/git-clone.git) and maintain it through the `alauda-vx.xx.xx` branch.

We use [renovate](https://gitlab-ce.alauda.cn/devops/tech-research/renovate/-/blob/main/docs/quick-start/0002-quick-start.md) to automatically fix vulnerabilities in corresponding versions.

## Repository Structure

Based on the original code, the following content has been added:

- [alauda-auto-tag.yaml](./.github/workflows/alauda-auto-tag.yaml): Automatically tags and triggers goreleaser when a PR is merged into the `alauda-vx.xx.xx` branch
- [release-alauda.yaml](./.github/workflows/release-alauda.yaml): Supports tag updates or manual triggering of goreleaser (this pipeline is not triggered when tags are automatically created in actions, as actions are designed not to recursively trigger multiple actions)
- [reusable-release-alauda.yaml](./.github/workflows/reusable-release-alauda.yaml): Executes goreleaser to create releases
- [scan-alauda.yaml](.github/workflows/scan-alauda.yaml): Performs trivy vulnerability scans (`rootfs` scans go binary)
- [.goreleaser-alauda.yml](image/git-init/.goreleaser-alauda.yml): Configuration file for releasing alauda versions

## Special Modifications

1. [.goreleaser-alauda.yml](image/git-init/.goreleaser-alauda.yml) is located in the build directory `image/git-init`
2. The trigger condition for [build.yaml](.github/workflows/build.yaml) has been added with the `alauda-v*` branch

## Pipelines

### Triggered When Submitting a PR

- [build.yaml](.github/workflows/build.yaml): Official testing pipeline, including unit tests, integration tests, etc.

### Triggered When Merging into the alauda-vx.xx.xx Branch

- [alauda-auto-tag.yaml](.github/workflows/alauda-auto-tag.yaml): Automatically tags and triggers goreleaser
- [reusable-release-alauda.yaml](.github/workflows/reusable-release-alauda.yaml): Executes goreleaser to create releases (triggered by `alauda-auto-tag.yaml`)

### Scheduled or Manual Triggering

- [scan-alauda.yaml](.github/workflows/scan-alauda.yaml): Performs trivy vulnerability scans (`rootfs` scans go binary)

### Others

Other officially maintained pipelines have not been modified, and some irrelevant pipelines have been disabled on the Action page.

## Renovate Vulnerability Fixing Mechanism

The renovate configuration file is [renovate.json](https://github.com/AlaudaDevops/trivy/blob/main/renovate.json)

1. renovate detects vulnerabilities in branches and submits PRs for fixes
2. PRs automatically run tests
3. After all tests pass, renovate automatically merges the PR
4. After the branch is updated, an action automatically tags (e.g., v0.62.1-alauda-0, both patch version and the last digit will increment)
5. goreleaser automatically publishes releases based on tags

## Maintenance Plan

When upgrading to a new version, follow these steps:

1. Create an alauda branch from the corresponding tag, for example, the `v0.62.1` tag corresponds to the `alauda-v0.62.1` branch
2. Cherry-pick previous alauda branch changes to the new branch and push

Renovate automatic fixing mechanism:
1. After renovate submits a PR, pipelines will automatically run; if all tests pass, the PR will be automatically merged
2. After merging into the `alauda-v0.62.1` branch, goreleaser will automatically create a `v0.62.2-alauda-0` release (note: not `v0.62.1-alauda-0`, because upgrading the version allows renovate to recognize it)
3. renovate configured in other plugins will automatically fetch artifacts from releases based on configuration
